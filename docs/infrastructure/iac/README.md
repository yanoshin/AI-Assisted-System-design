# IaC 実装指示書（雛形）

Terraform と Ansible の **役割分担・ディレクトリ構成・命名規則** をまとめた雛形。コードそのものではなく、IaC を書く人がこれを読めば構造を再現できるようにしている。

## ファイル

| ファイル | 内容 |
|---|---|
| [`terraform.md`](terraform.md) | Terraform 雛形（プロビジョニング担当） |
| [`ansible.md`](ansible.md) | Ansible 雛形（構成管理・デプロイ担当） |

## Terraform と Ansible の使い分け

| 対象 | Terraform | Ansible |
|---|---|---|
| クラウドリソースの作成（VPC / VM / RDS / IAM） | ◎ | △（プラグインで可だが非推奨） |
| インスタンス内 OS 設定（パッケージ / ユーザー / ファイル） | × | ◎ |
| デプロイ（アプリ配置 / 起動 / ローリング更新） | △（外部ツール連携） | ◎ |
| 状態管理（差分検出・依存解決） | ◎（state） | △（冪等性に依存） |
| シークレット適用 | △（state に出る） | ◎（Vault / SOPS と連携） |
| Kubernetes リソース | △ (kubernetes provider) | △ (k8s モジュール) |

**推奨**: **Terraform でプロビジョニング → Ansible で構成管理・デプロイ** の2段階。コンテナベースなら Ansible の役割は最小化される（イメージ build + 差し替えで完結）。

## 全体の流れ

```
1. 設計書 (design.md) → service-mapping.md で具体サービス決定
        ↓
2. Terraform で基盤プロビジョニング（VPC / DB / IAM / Secrets 等）
        ↓
3. (VM 構成の場合) Ansible で OS 構成・アプリ配置
   (コンテナ構成の場合) CI でイメージ build → コンテナサービスに deploy
        ↓
4. 監視・アラート設定（Terraform で同居 or 別リポジトリ）
```

## リポジトリ配置

本リポジトリは **設計ドキュメント集** なので、実 IaC コードは別リポジトリで管理することを推奨。

```
infra-iac/                # 実装リポジトリ（別）
├── terraform/
└── ansible/

ai-system-design/         # 本リポジトリ（設計）
└── docs/infrastructure/iac/  ← terraform.md / ansible.md
```

## 関連

- 採用 IaC の決定: [`../interview.md` Q-IAC-01](../interview.md#q-iac-01-iac-ツールは)
- 抽象 → 具体の対照: [`../service-mapping.md`](../service-mapping.md)
- 既存設計の方針: [詳細設計書 §8.1 IaC 構成](../../detail-design/詳細設計書.md#81-iac-構成)
