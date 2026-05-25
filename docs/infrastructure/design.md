# インフラ設計

[`interview.md`](interview.md) の回答を反映して、採用するインフラ設計を記入する。**未確定の章は空欄のまま**、決まった順に埋めていく。

## メタ情報

| 項目 | 値 |
|---|---|
| 対象プロジェクト | （例: Daily Diary） |
| 記入者 | |
| 最終更新 | YYYY-MM-DD |
| 採用 IaC | （Terraform / Ansible / 両方 / その他） |

---

## 1. 全体方針

- ホスティング種別（Q-HOST-01）: 
- 採用クラウド / オンプレ（Q-HOST-02）: 
- リージョン（Q-HOST-03）: 
- スケーリング戦略（Q-SCALE-01）: 
- RTO / RPO（Q-DR-01）: 

> 抽象→具体の対応は [`service-mapping.md`](service-mapping.md) を参照。

---

## 2. コンピュート

- コンピュート種別（Q-COMP-01）: 
- 配信構成（Q-COMP-02）: 
- 採用サービス: 
  - Web 配信: 
  - API: 
  - バッチ / ジョブ: 

例:
```
- 採用: コンテナ (マネージド) → AWS の場合 ECS on Fargate / GCP の場合 Cloud Run
- Web (SSR): 同上
- API:      同上
- ジョブ:    EventBridge + Step Functions / Cloud Scheduler + Cloud Run jobs
```

---

## 3. ネットワーク

- ネットワーク分離（Q-NET-01）: 
- 公開範囲（Q-NET-02）: 
- WAF / DDoS（Q-NET-03）: 

採用構成:
```
- VPC: CIDR 10.0.0.0/16
  - パブリックサブネット: CDN/LB（10.0.0.0/24, 10.0.1.0/24）
  - プライベートサブネット: Compute（10.0.10.0/24, 10.0.11.0/24）
  - データサブネット: DB（10.0.20.0/24, 10.0.21.0/24）
- SG:
  - LB → App: HTTPS (443)
  - App → DB: 5432
  - Egress: 必要最小限（OIDC IdP のみ）
```

構成図テンプレ → [`architecture/network-template.mmd`](architecture/network-template.mmd) をコピーして編集。

---

## 4. データ層

- データ配置（Q-DATA-01）: 
- バックアップ（Q-DATA-02）: 
- 採用 DB: 
- マイグレーション方式: 

例:
```
- DB: PostgreSQL 16 マネージド
- バックアップ: 1時間毎スナップショット、世代7日、別リージョンへ複製
- マイグレーション: prisma migrate (CI から本番 DB に apply)
```

---

## 5. シークレット・監査

- シークレット管理（Q-SEC-01）: 
- 監査ログ（Q-SEC-02）: 
- 暗号化（保管時 / 通信時）: 

例:
```
- シークレット: クラウドマネージド (Secrets Manager / Secret Manager / Key Vault)
- 監査: クラウド標準（CloudTrail / Audit Logs）に転送、保管1年
- 暗号化: TLS 1.2+ (通信)、AES-256 (DB at-rest)
```

---

## 6. 監視・ログ

- 集約先（Q-OBS-01）: 
- メトリクス: 
- ログ: 
- アラート通知先: 

---

## 7. CI/CD

- デプロイ経路（Q-CICD-01）: 
- 環境一覧: dev / stg / prd
- 承認フロー: 
- ロールバック方針: 

---

## 8. 障害対応

- ランブック保管先: 
- オンコール体制: 
- インシデント分類（P1 / P2 / P3）: 

例:
```
- ランブック: docs/operations/runbook.md (将来)
- オンコール: PagerDuty
- P1: 全停止、即時対応
- P2: 一部機能停止、1時間以内
- P3: 個別不具合、翌営業日
```

---

## 9. 構成図

`architecture/` 配下のテンプレをコピーして、本プロジェクト用の構成図を作る。

- ネットワーク図: `architecture/network-template.mmd` → `architecture/network.mmd`
- 配置図: `architecture/deployment-template.mmd` → `architecture/deployment.mmd`

---

## 10. IaC

- 採用ツール（Q-IAC-01）: 
- リポジトリ構成: （別リポジトリ推奨）
- 詳細: [`iac/`](iac/) を参照

---

## 関連ドキュメント

- [基本設計書 §7 インフラ/デプロイ基本設計](../basic-design/基本設計書.md)
- [詳細設計書 §8 インフラ詳細](../detail-design/詳細設計書.md)
- [ADR](../decisions/)
