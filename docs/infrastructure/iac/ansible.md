# Ansible 実装指示書（雛形）

Ansible を使った **構成管理・デプロイ** の雛形。VM ベースの構成を想定し、コンテナベースの場合は本ファイルの範囲は最小化される（イメージ build + 差し替えに置き換わる）。

## 推奨ディレクトリ構成

```
infra-iac/ansible/
├── README.md
├── ansible.cfg                 # inventory_path / roles_path / vault 設定
├── requirements.yml            # 依存ロール / collections
├── inventories/                # 環境別インベントリ
│   ├── dev/
│   │   ├── hosts.yml           # 動的インベントリ推奨（クラウド API から取得）
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   ├── web.yml
│   │   │   └── db.yml
│   │   └── host_vars/
│   ├── stg/
│   └── prd/
├── playbooks/                  # ユースケース別
│   ├── site.yml                # 全構成（初期構築）
│   ├── deploy_app.yml          # アプリのデプロイのみ
│   ├── rotate_secrets.yml      # シークレットローテ
│   └── update_packages.yml     # OS パッチ適用
├── roles/                      # 役割別
│   ├── common/                 # 全ホスト共通（ユーザー / ssh / sysctl）
│   ├── webapp/                 # アプリ配置・systemd unit
│   ├── reverse_proxy/          # Nginx / Caddy 設定
│   ├── postgres_client/        # DB クライアント設定
│   ├── observability_agent/    # ログ / メトリクスエージェント
│   └── security_hardening/     # firewall / fail2ban / unattended-upgrades
└── vars/
    └── vault/                  # Ansible Vault 暗号化された機密値
```

## ロール作法

- **1ロール = 1責務**。複合的になったら分割
- ファイル構成: `tasks/main.yml` / `handlers/main.yml` / `templates/` / `files/` / `defaults/main.yml` / `vars/main.yml` / `meta/main.yml` / `README.md`
- **冪等性を必ず確保**: `state: present` を使い、`shell` / `command` の濫用を避ける
- ハンドラ（restart 等）は **`notify` で発火**、タスク末尾で直接 restart しない
- 変数は `defaults/` に書いて上書きしやすく。`vars/` は固定値のみ

## 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| ロール | `lower_snake_case` | `webapp` |
| プレイブック | `動詞_目的.yml` | `deploy_app.yml` |
| 変数 | `lower_snake_case` | `app_port` |
| 暗号化変数 | `vault_*` プレフィックス | `vault_db_password` |
| ホストグループ | 役割で | `web`, `db`, `bastion` |
| タグ | 役割で付与 | `common`, `deploy`, `security` |

## インベントリ

### 動的インベントリ推奨
クラウド API からホストを取得（タグでグルーピング）。
- AWS: `amazon.aws.aws_ec2`
- GCP: `google.cloud.gcp_compute`
- Azure: `azure.azcollection.azure_rm`

### 静的の場合は YAML 形式
```yaml
all:
  children:
    web:
      hosts:
        web-prd-1: { ansible_host: 10.0.10.11 }
        web-prd-2: { ansible_host: 10.0.10.12 }
    db:
      hosts:
        db-prd-1: { ansible_host: 10.0.20.11 }
```

## 認証情報の扱い

- **平文の機密値は絶対にコミットしない**
- 暗号化手段（いずれか）:
  - **Ansible Vault** — Ansible 標準
  - **HashiCorp Vault** — マルチクラウド・動的シークレット
  - **SOPS** — ファイル単位の暗号化、KMS と連携
- パスワードは `become_password` 等を介して環境変数で渡す（CI から）

## デプロイ戦略

| 戦略 | 適合 | Ansible での実現 |
|---|---|---|
| In-Place | 開発・小規模 | `serial: 1` で1台ずつ |
| Rolling | 本番標準 | `serial: 33%` + ヘルスチェック |
| Blue/Green | 安全重視 | 別グループに配置 → LB 切替 |
| Canary | 段階的検証 | `serial` を段階的に拡大 |

### ローリングデプロイの基本形（疑似コード）

```yaml
- name: Deploy web app (rolling)
  hosts: web
  serial: 33%
  pre_tasks:
    - name: Drain from load balancer
      # LB の API を叩いて該当ホストを除外
  roles:
    - webapp
  post_tasks:
    - name: Wait for health check
      uri: { url: "http://{{ inventory_hostname }}:8080/health" }
      register: result
      until: result.status == 200
      retries: 30
      delay: 5
    - name: Add back to load balancer
```

## CI での実行

```
PR 作成 → ansible-lint + yamllint + 文法チェック
     ↓
PR レビュー
     ↓
main マージ → dev に site.yml apply
     ↓
タグ → 手動承認 → stg apply → 監視 → prd apply
```

## ロール一覧（雛形）

### `roles/common/`
- 標準ユーザー / SSH 鍵 / sudoers
- タイムゾーン / NTP / journald
- 基本パッケージ（curl / jq / git 等）

### `roles/webapp/`
- アプリのソース取得（Git or アーティファクト）
- 依存インストール、ビルド
- systemd unit の配置と enable
- ヘルスチェック

### `roles/reverse_proxy/`
- Nginx インストール
- TLS 証明書配置（acme.sh / certbot 連携）
- リバースプロキシ設定（テンプレート）

### `roles/postgres_client/`
- psql / pg_dump
- 接続情報（envファイル）配置

### `roles/observability_agent/`
- メトリクス agent（node_exporter / CloudWatch Agent）
- ログ転送（Vector / Fluent Bit）

### `roles/security_hardening/`
- ufw / firewalld 設定
- fail2ban
- unattended-upgrades
- SSH ポリシー強化

## やってはいけないこと

- `shell` / `command` を冪等性なしで使う
- 平文パスワードを変数ファイルにコミット
- 1つのロールに無関係なタスクを詰め込む
- 本番で `--check` なしの「初回実行」（必ず `--check --diff` で差分確認 → apply）
- `become: yes` をプレイブック全体でデフォルトに（必要なタスクだけ）

## コンテナ採用時の補足

コンテナベースなら **Ansible の役割は最小化** される。残る用途:
- ホスト OS の最低限のセットアップ（systemd / Docker 自体のインストール）
- Bastion ホストの管理
- 緊急時の構成変更

それ以外はコンテナイメージで完結させる方が運用が楽。

## 関連

- [`terraform.md`](terraform.md) — プロビジョニング側
- [`../service-mapping.md`](../service-mapping.md) — 抽象 → クラウド具体
- [`../interview.md` Q-CICD-01](../interview.md#q-cicd-01-デプロイ経路は) — CI/CD 経路
