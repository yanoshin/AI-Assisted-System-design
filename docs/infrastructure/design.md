# インフラ設計 — Daily Diary

[`interview.md`](interview.md) の回答を反映した、Daily Diary の採用インフラ設計。抽象→具体の対応は [`service-mapping.md`](service-mapping.md) を参照。

## メタ情報

| 項目 | 値 |
|---|---|
| 対象プロジェクト | Daily Diary |
| 記入者 | （指示者 + Claude Code によるインタビュー） |
| 最終更新 | 2026-05-26 |
| 採用 IaC | Terraform + Ansible（Q-IAC-01） |

---

## 1. 全体方針

| 項目 | 採用 | 出典 Q-ID |
|---|---|---|
| ホスティング種別 | パブリッククラウド単独 | Q-HOST-01 |
| 採用クラウド | **GCP (Google Cloud)** | Q-HOST-02 |
| リージョン | **asia-northeast1（東京）単一** | Q-HOST-03 |
| スケーリング戦略 | オートスケール（水平） | Q-SCALE-01 |
| RTO / RPO | RTO 4h / RPO 1h | Q-DR-01 |

> 抽象 → GCP サービス名の対応は [`service-mapping.md`](service-mapping.md) の GCP 列を参照。

---

## 2. コンピュート

| 項目 | 採用 |
|---|---|
| コンピュート種別（Q-COMP-01） | コンテナ（マネージド） |
| 配信構成（Q-COMP-02） | 同一サーバ（Web/API を1プロセスで） |
| 採用サービス | **Cloud Run** |

詳細:
- **Web (SSR) + API** を1コンテナイメージで配信（Next.js のサーバが API もホストする想定）
- Cloud Run のオートスケール（最小0、最大10程度から開始）
- バッチ / ジョブ: **Cloud Scheduler + Cloud Run jobs**（BAT-01 セッション掃除用、詳細設計書 §5.1）

---

## 3. ネットワーク

| 項目 | 採用 | 出典 |
|---|---|---|
| ネットワーク分離（Q-NET-01） | VPC + パブリック/プライベートサブネット | |
| 公開範囲（Q-NET-02） | **API も直接公開（CDN なし）** | MVP コスト最適化 |
| WAF / DDoS（Q-NET-03） | **不要 / 最小限** | MVP コスト最適化 |

採用構成（東京リージョン asia-northeast1）:
```
- VPC: dailydiary-vpc (CIDR 10.10.0.0/16)
  - パブリックサブネット: 10.10.0.0/24（LB 配置用）
  - プライベートサブネット: 10.10.10.0/24（Cloud Run 用）※サーバレス VPC アクセスコネクタ
  - データサブネット: 10.10.20.0/24（Cloud SQL Private IP）
- Firewall:
  - LB → Cloud Run: 内部経路（Serverless NEG）
  - Cloud Run → Cloud SQL: 5432（Private IP 経由）
  - Egress: 必要最小限（accounts.google.com への HTTPS のみ）
```

> **既存設計書との不整合の注記**:
> - 基本設計書 §8.3・詳細設計書 §8.2 では WAF / CDN を推奨していたが、本インタビューでは **MVP コスト最適化のため不採用** と決定。
> - 利用者数増 / セキュリティ要件強化のタイミングで [`docs/decisions/`](../decisions/) に ADR として再評価を記録する想定。

構成図 → [`architecture/network.mmd`](architecture/network.mmd)

---

## 4. データ層

| 項目 | 採用 |
|---|---|
| データ配置（Q-DATA-01） | マネージド DB を VPC 内（Cloud SQL Private IP） |
| バックアップ（Q-DATA-02） | 1時間毎スナップショット、世代7日 |
| 採用 DB | **Cloud SQL for PostgreSQL 16** |
| マイグレーション方式 | CI から `prisma migrate` 等で本番 DB に apply |

詳細:
- インスタンスタイプ: db-custom-2-7680（2vCPU/7.5GB から開始、要計測）
- 高可用性（HA）: MVP は単一ゾーンで開始、利用者数増で HA 化を検討
- バックアップ: 自動バックアップ ON、保存期間 7日、別リージョン複製は今回は **なし**
- 接続: Private IP のみ。Public IP は **オフ**

---

## 5. シークレット・監査

| 項目 | 採用 |
|---|---|
| シークレット管理（Q-SEC-01） | **Secret Manager** |
| 監査ログ（Q-SEC-02） | Cloud Audit Logs を保管 1年 |
| 暗号化（保管時 / 通信時） | TLS 1.2+ / Cloud SQL at-rest 暗号化（GCP デフォルト） |

シークレット一覧（Secret Manager で管理）:
- `DB_PASSWORD` — Cloud SQL アプリ用ユーザのパスワード
- `OIDC_CLIENT_SECRET` — Google OIDC クライアントシークレット
- `SESSION_SIGNING_KEY` — セッショントークン署名用

Cloud Run のサービスアカウントに `roles/secretmanager.secretAccessor` を付与し、起動時に環境変数として注入。

---

## 6. 監視・ログ

| 項目 | 採用 |
|---|---|
| 集約先（Q-OBS-01） | **Cloud Monitoring + Cloud Logging** |
| メトリクス | Cloud Run のリクエスト数 / レイテンシ / エラー率、Cloud SQL の CPU / 接続数 |
| ログ | Cloud Run / Cloud SQL の標準ログを Cloud Logging に集約 |
| アラート通知先 | Cloud Monitoring → Slack（webhook 連携） |

主要アラート:
- API エラー率 5分平均 1% 超 → Slack
- API レイテンシ p95 1秒超 → Slack
- DB 接続失敗 1分以内1回以上 → Slack（将来 PagerDuty 連携検討）

---

## 7. CI/CD

| 項目 | 採用 |
|---|---|
| デプロイ経路（Q-CICD-01） | CI → コンテナレジストリ → 本番 |
| 使用ツール | **Cloud Build → Artifact Registry → Cloud Run** |
| 環境一覧 | dev / stg / prd |
| 承認フロー | dev 自動、stg 自動、prd は手動承認 |
| ロールバック方針 | Cloud Run のリビジョントラフィック切替（前リビジョンに 100% 戻す） |

CI トリガ:
- PR 作成 → Cloud Build で lint / test / build （image push なし）
- main マージ → Cloud Build で image build + push → dev へ自動 deploy
- タグ push（`v*`） → stg → prd（手動承認）

---

## 8. 障害対応

| 項目 | 採用 |
|---|---|
| ランブック保管先 | `docs/operations/runbook.md`（将来作成） |
| オンコール体制 | Slack 通知 → 開発チームで対応（MVP は専任オンコールなし） |
| インシデント分類 | P1（全停止・1時間以内）/ P2（一部機能停止・翌営業日）/ P3（個別不具合・週次まとめ対応） |

ローテーションが必要になった段階で PagerDuty 等の専用ツール導入を再評価。

---

## 9. 構成図

- ネットワーク図: [`architecture/network.mmd`](architecture/network.mmd)
- 配置図: [`architecture/deployment.mmd`](architecture/deployment.mmd)

---

## 10. IaC

| 項目 | 採用 |
|---|---|
| 採用ツール（Q-IAC-01） | **Terraform + Ansible** |
| リポジトリ構成 | 別リポジトリ推奨（例: `dailydiary-infra`） |

役割分担:
- **Terraform**: GCP プロビジョニング（VPC / Cloud Run / Cloud SQL / Secret Manager / Cloud Build トリガ / IAM）
- **Ansible**: 最小限の用途のみ（必要時の踏み台ホスト構成、Cloud SQL Auth Proxy のセットアップなど）。コンテナ中心構成のため通常は不要

雛形は [`iac/terraform.md`](iac/terraform.md) と [`iac/ansible.md`](iac/ansible.md) を参照。

---

---

## 11. 環境構成（個別開発者 / dev / stg / prd）

| 項目 | 採用 | 出典 Q-ID |
|---|---|---|
| 用意する環境（Q-ENV-01） | **個別開発者環境 + dev + stg + prd** | Q-ENV-01 |
| 分離レベル（Q-ENV-02） | **同一 GCP プロジェクト内で命名プレフィックスで分離** | Q-ENV-02 |
| ステージングデータ（Q-ENV-03） | **空 / 開発時に投入（合成データ）** | Q-ENV-03 |
| フィーチャーフラグ（Q-ENV-04） | **不要（MVP）** | Q-ENV-04 |

> **注**: Q-ENV-02 でプロジェクト分離（B）ではなく名前空間分離（A）を採用。**コスト・運用シンプル化** が優先。利用者数増・コンプライアンス強化のタイミングでプロジェクト分離への昇格を再評価する（ADR 化候補）。

各環境の詳細:

| 環境 | 用途 | スケール | データ | 認証 IdP（Google OIDC クライアント） | 命名プレフィックス |
|---|---|---|---|---|---|
| **個別開発者** | 各開発者の手元検証用 | 最小 / オンデマンド | 合成データ | dev 用クライアント共用 | `dev-{handle}-*`（例: `dev-yanoshin-cloudrun`） |
| **dev** | チーム共有の検証 | 最小（min=0） | 合成データ | dev 用クライアント | `dev-*` |
| **stg** | 受入・リリース前検証 | 本番に近い（min=1） | **空 → 開発時に合成データ投入** | stg 用クライアント | `stg-*` |
| **prd** | 利用者向け本番 | オートスケール（min=1, max=10） | 利用者の実データ | prd 用クライアント | `prd-*` |

GCP リソースの命名規則:
```
{env}-{role}        例: prd-cloudrun-app, stg-cloudsql-main, dev-yanoshin-cloudrun
```

環境別の Cloud SQL インスタンス分離:
- prd: 専用 Cloud SQL インスタンス（HA は将来検討）
- stg: 専用 Cloud SQL インスタンス（最小スペック）
- dev: 共用 Cloud SQL インスタンス + データベース分離（または個別開発者はローカル Docker）
- 個別開発者環境: 原則 **ローカル Docker Compose**（次項 §12）を使い、必要時のみ dev クラウドへ接続

環境固有の値（CIDR・接続先・OIDC クライアントID）は IaC の `envs/<env>/` で管理（[`iac/terraform.md`](iac/terraform.md) 参照）。

---

## 12. ローカル開発環境

| 項目 | 採用 | 出典 Q-ID |
|---|---|---|
| 動かす範囲（Q-LOCAL-01） | **すべてローカル**（Docker Compose で DB 含む） | Q-LOCAL-01 |
| ローカル DB（Q-LOCAL-02） | **Docker Compose（PostgreSQL 16 公式イメージ）** | Q-LOCAL-02 |
| 認証（Q-LOCAL-03） | **本物の Google OIDC（dev 用クライアント）** | Q-LOCAL-03 |
| シークレット（Q-LOCAL-04） | **`.env.local`（gitignore）+ `.env.example`** | Q-LOCAL-04 |
| セットアップ自動化（Q-LOCAL-05） | **README + Makefile** | Q-LOCAL-05 |

### 12.1 セットアップ手順（Daily Diary）

新規参加者の手順は README の「Getting Started」または別途 `docs/development/setup.md` にまとめる。最低限以下を含める。

```
1. リポジトリを clone
2. 必要なツールをインストール
   - Node.js 20 (mise / asdf 推奨)
   - Docker Desktop
   - gcloud CLI (本番接続が必要なときのみ)
3. シークレットの取得
   - cp .env.example .env.local
   - チームの 1Password / Slack DM 等で .env.local の値を埋める
     （特に GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, SESSION_SIGNING_KEY）
4. ローカル DB の起動
   - make db-up         # docker compose up -d postgres
5. マイグレーション + シード
   - make db-migrate    # prisma migrate deploy（or 採用 ORM のコマンド）
   - make db-seed       # テストユーザー + サンプル日記を投入
6. 開発サーバ起動
   - make dev           # next dev（Web + API 同居）
7. ブラウザで http://localhost:3000 を開いて Google でログイン
```

### 12.2 ローカル ⇄ クラウドの境界

| 要素 | ローカル | dev クラウド | stg / prd |
|---|---|---|---|
| アプリ | ローカル `next dev`（Web+API 同居） | Cloud Run（dev-*） | Cloud Run（stg-* / prd-*） |
| DB | Docker Compose `postgres:16` | Cloud SQL（共用 dev インスタンス） | Cloud SQL（環境別） |
| 認証 IdP | 本物 Google OIDC（dev クライアント） | 本物 Google OIDC（dev クライアント） | 本物 Google OIDC（stg / prd 各クライアント） |
| シークレット | `.env.local` | Secret Manager（`dev-*`） | Secret Manager（`stg-*` / `prd-*`） |

> dev 用 OAuth クライアントは「ローカル開発」と「dev クラウド環境」で共用。リダイレクト URI に `http://localhost:3000/api/auth/callback` と `https://dev.dailydiary.example.com/api/auth/callback` を両方登録する。

### 12.3 整備するファイル雛形（リポジトリ直下）

| ファイル | 内容 |
|---|---|
| `.env.example` | `GOOGLE_CLIENT_ID=` / `GOOGLE_CLIENT_SECRET=` / `DATABASE_URL=postgres://...` / `SESSION_SIGNING_KEY=` のキー一覧（値は空） |
| `.gitignore` | `.env.local`, `.env.*.local` を追加 |
| `docker-compose.yml` | サービス `postgres`（image: `postgres:16`、port 5432、volume で永続化） |
| `Makefile` | `dev` / `db-up` / `db-down` / `db-migrate` / `db-seed` / `test` / `lint` のターゲット |
| `README.md` | 上記「Getting Started」を掲載 |

### 12.4 トラブル対応の常套句

| 症状 | 対処 |
|---|---|
| `make db-up` で port 5432 競合 | ローカルに PostgreSQL が常駐していないか確認 (`lsof -i :5432`)。あれば停止 or compose 側のポート変更 |
| OIDC ログインでリダイレクト失敗 | Google Console の OAuth クライアントの **承認済みリダイレクト URI** に `http://localhost:3000/api/auth/callback` が登録されているか確認 |
| `.env.local` の値を更新したのに反映されない | 開発サーバを再起動（Next.js は環境変数の変更でホットリロードしない） |

---

## 関連ドキュメント

- [tech-stack/decision-record.md](../tech-stack/decision-record.md) — アプリ層の選定結果
- [基本設計書 §7 インフラ/デプロイ基本設計](../basic-design/基本設計書.md)
- [詳細設計書 §8 インフラ詳細](../detail-design/詳細設計書.md)
- [ADR](../decisions/)

## ADR 化を推奨する決定

- **クラウド選定（GCP）**: 大粒度・将来影響大 → ADR 0004 として独立記録を推奨
- **WAF / CDN 不採用**: 既存設計書の方針との不整合を残すため、ADR で明示が望ましい
