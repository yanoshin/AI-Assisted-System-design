# インフラ設計パッケージ

アプリケーションを動かす **インフラ（実行環境）** を設計する雛形です。クラウド (AWS / GCP / Azure) とオンプレの両方に対応できるよう、抽象名で書いて [`service-mapping.md`](service-mapping.md) で具体実装に翻訳します。

## 進め方

```
1. interview.md           … 質問に答えてホスティング / コンピュート等を決める
        ↓
2. design.md              … 採用設計を記入（章ごとに）
        ↓
3. service-mapping.md     … 抽象名を選択したクラウド/オンプレの具体名へ
        ↓
4. architecture/*.mmd     … 構成図テンプレをコピーして実設計図に
        ↓
5. iac/                   … 実装指示書（Terraform / Ansible 雛形）に従って IaC を書く
```

### Claude にインタビューを進めてもらう場合
- 「**infrastructure のインタビューを進めて**」と依頼
- Claude は `interview.md` を読んで `AskUserQuestion` で順に確認し、`design.md` / `architecture/*.mmd` に逐次反映
- ワークフローは [CLAUDE.md](../../CLAUDE.md) を参照

## ファイル構成

| ファイル / フォルダ | 役割 |
|---|---|
| [`interview.md`](interview.md) | 質問集（ホスティング / コンピュート / ネットワーク / データ / DR / 監視 / シークレット / CI-CD / IaC） |
| [`design.md`](design.md) | 採用設計の記入欄テンプレ |
| [`service-mapping.md`](service-mapping.md) | 抽象 → AWS / GCP / Azure / オンプレ の対照表 |
| [`architecture/`](architecture/) | 構成図テンプレ（ネットワーク / 配置） |
| [`iac/`](iac/) | Terraform / Ansible 実装指示書の雛形 |

## 既存ドキュメントとの関係

| 既存 | このパッケージでの扱い |
|---|---|
| [基本設計書 §7](../basic-design/基本設計書.md#7-インフラ--デプロイ基本設計) | 方針のみ。**設計プロセスと対照表** はこちら |
| [詳細設計書 §8](../detail-design/詳細設計書.md#8-インフラ詳細) | 詳細はこちらの `iac/` を参照 |
| [ADR 0001 アーキテクチャ](../decisions/0001-architecture.md) | アプリ層との接続の前提 |

## 関連: 技術スタック

アプリ実装側の選定は [`../tech-stack/`](../tech-stack/) で扱う。技術スタック → アプリ、本パッケージ → 実行環境、と分担する。
