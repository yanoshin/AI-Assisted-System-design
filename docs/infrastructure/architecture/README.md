# インフラ構成図テンプレート

抽象名で書かれたネットワーク図と配置図のテンプレート。コピーしてプロジェクト用に編集する。

## ファイル

| ファイル | 用途 | コピー先の命名 |
|---|---|---|
| [`network-template.mmd`](network-template.mmd) | ネットワーク構成図（VPC / サブネット / SG） | `network.mmd` |
| [`deployment-template.mmd`](deployment-template.mmd) | 配置図（CDN / Compute / DB / IdP / 監視） | `deployment.mmd` |

## 使い方

1. テンプレを同ディレクトリにコピー: `cp network-template.mmd network.mmd`
2. クラウド選定が決まっていれば [`../service-mapping.md`](../service-mapping.md) を引いて、ノード名を具体サービスに置き換える
3. リージョン / アベイラビリティゾーン / CIDR / セキュリティグループの具体値を埋める

## 描画

Mermaid のため、GitHub 上でそのまま描画されます。ローカル描画は VS Code 拡張「Mermaid Preview」等で。

## 補足

- 大規模化したら PlantUML の `!include` で部分図を分割するのも可（[basic-design/data-model](../../basic-design/data-model/) の PlantUML 構造を参考に）
- 構成変更時は **必ず本図と Terraform を同期** すること（差分の検知は手作業）
