# Terraform 実装指示書（雛形）

Terraform を使った **プロビジョニング** の雛形。コードのコピペではなく、ディレクトリ構成・命名規則・モジュール作法の指示。

## 推奨ディレクトリ構成

```
infra-iac/terraform/
├── README.md
├── modules/                  # 再利用可能なモジュール
│   ├── network/              # VPC / サブネット / SG / ルートテーブル
│   ├── compute/              # コンピュート（VM / コンテナ / 関数）
│   ├── database/             # RDB / NoSQL
│   ├── secrets/              # シークレット管理
│   ├── iam/                  # IAM / サービスアカウント
│   ├── observability/        # ログ / メトリクス / アラート
│   └── cicd/                 # CI/CD 関連リソース
├── envs/                     # 環境別の構成
│   ├── dev/
│   │   ├── main.tf           # modules 呼び出し
│   │   ├── terraform.tfvars  # 環境固有値
│   │   ├── backend.tf        # state バックエンド設定
│   │   └── providers.tf      # プロバイダ + バージョン固定
│   ├── stg/
│   └── prd/
└── shared/                   # 全環境共通の定数 / data source
```

## モジュール作法

- **1モジュール = 1ドメイン**（network / compute / database 等）。クラウド固有はリソース名で吸収し、入出力（variables / outputs）は抽象的に保つ
- 各モジュールは `main.tf` / `variables.tf` / `outputs.tf` / `versions.tf` / `README.md` を持つ
- **入力**: `var.*`（必須・任意・型・description を必ず）
- **出力**: 他モジュールが使う ID・エンドポイント・名前のみ。内部状態は出さない
- **副作用は持たない**: ローカル `exec` / `provisioner` は避ける（Ansible 側に寄せる）

### 例: `modules/network/variables.tf` の抽象構造

```hcl
# 入力（抽象的に）
variable "vpc_cidr" {
  type        = string
  description = "VPC の CIDR ブロック (例: 10.0.0.0/16)"
}
variable "public_subnet_cidrs" {
  type        = list(string)
  description = "パブリックサブネットの CIDR リスト"
}
variable "private_subnet_cidrs" {
  type        = list(string)
  description = "プライベートサブネットの CIDR リスト"
}
variable "availability_zones" {
  type        = list(string)
  description = "AZ リスト"
}
```

> 実コードは別リポジトリで書く。本ファイルは「こういう構造で書く」ことの指示。

## 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| リソース | `{service}-{role}-{env}` | `app-web-prd` |
| タグ | `Project` / `Environment` / `ManagedBy=terraform` | 全リソースに必須 |
| ファイル | `snake_case.tf` | `network.tf` |
| 変数 / 出力 | `lower_snake_case` | `vpc_cidr` |
| モジュール | 単数形 | `module "network"` |
| state ファイル | `{env}/terraform.tfstate` | `prd/terraform.tfstate` |

## state 管理

- **ローカル state は使わない**。常にリモートバックエンド
- 推奨:
  - AWS: S3 + DynamoDB lock
  - GCP: GCS + native lock
  - Azure: Blob Storage + native lock
  - オンプレ: Terraform Cloud / Atlantis / minio + Consul lock
- 環境ごとに **state を分割**（dev / stg / prd で別ファイル）
- production の state は **読み取り制限** を厳密に

## バージョン管理

- `versions.tf` で Terraform 本体・プロバイダのバージョンを固定（`required_version`, `required_providers`）
- `.terraform.lock.hcl` をコミット
- 月1回はバージョン見直し（パッチ更新の取り込み）

## CI での適用フロー

```
PR 作成 → fmt + validate + tflint → terraform plan (環境別)
     ↓
PR レビュー（plan 出力を必ず添付）
     ↓
main マージ → 自動 apply (dev)
     ↓
タグ push → 手動承認 → 自動 apply (stg → prd)
```

## モジュール一覧（雛形）

### `modules/network/`
- VPC、サブネット（public / private / data の3層）、ルートテーブル、IGW / NAT、SG、フローログ
- 出力: `vpc_id`, `subnet_ids`（用途別）, `sg_ids`

### `modules/compute/`
- 抽象: コンピュート種別を `var.compute_type`（"vm" / "container" / "serverless"）で分岐
- 各クラウドの具体リソースは `service-mapping.md` を参照
- 出力: `endpoint`, `target_group_arn`（LB ターゲット）

### `modules/database/`
- RDB（マルチ AZ、自動バックアップ、パラメータグループ）
- データサブネットに配置、SG は App からのみ許可
- 出力: `endpoint`, `port`, `secret_arn`（接続情報）

### `modules/secrets/`
- シークレット管理（KMS or Secret Manager）
- ローテーション設定（任意）
- 出力: `secret_arn` / `kms_key_arn`

### `modules/iam/`
- サービスごとの IAM ロール（最小権限）
- 「コンピュート → DB シークレット読取」のような粒度
- 出力: `role_arn` 群

### `modules/observability/`
- ログ集約先、メトリクスダッシュボード、アラート
- 通知先（Slack / PagerDuty）は変数化
- 出力: なし or `dashboard_url`

### `modules/cicd/`
- コンテナレジストリ、CI 用 IAM、デプロイ用権限
- 出力: `registry_url`, `ci_role_arn`

## やってはいけないこと

- リソースを `terraform import` せずに本番で直接作成してから取り込む（差分検知ができない）
- `count = 0` で削除（リソース置換と混同しやすい。`for_each` を推奨）
- 大量の `local` 計算でロジックを書く（外部スクリプト化）
- state を Git にコミット
- パスワード / トークンを `*.tf` に直書き（必ず `sensitive = true` + 外部参照）

## 関連

- [`../service-mapping.md`](../service-mapping.md) — 抽象 → クラウド具体
- [`../interview.md` Q-IAC-01](../interview.md#q-iac-01-iac-ツールは) — 採用決定
- [`ansible.md`](ansible.md) — 構成管理側の指示
