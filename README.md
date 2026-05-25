# AI-Assisted System Design — Daily Diary

AI（Claude Code）と協働して整備した、Web アプリケーション **「Daily Diary」** の設計ドキュメント集です。本リポジトリにはアプリケーションコードは含まれず、**要件定義 / 基本設計 / 詳細設計** の各ドキュメントと UML 図のソーステキストのみが置かれています。

## 題材

**Daily Diary** — 毎日の日記を投稿・編集できる SSO 対応 Web アプリケーション。

| 項目 | 内容 |
|---|---|
| 認証 | Google SSO 専用（OIDC Authorization Code Flow + PKCE） |
| 投稿 | 1日1エントリ制約（同一利用者は同じ日付に1件のみ） |
| 編集 | 過去日記は無期限で編集可能（MVP は編集履歴なし） |
| 可視性 | プライベートのみ（公開・共有機能なし） |
| 添付 | テキストのみ（画像・音声は MVP 対象外） |

詳細は [`docs/requirements/要件定義書.md`](docs/requirements/要件定義書.md) を参照。

## ドキュメント構成

設計工程の流れに沿って **要件定義 → 基本設計 → 詳細設計** の3階層で整理しています。

| 階層 | 答えるべき問い | エントリ |
|---|---|---|
| 【A】要件定義 | 何を作るか / なぜ作るか | [要件定義書](docs/requirements/要件定義書.md) |
| 【B】基本設計 | どう作るか（全体像・方式） | [基本設計書](docs/basic-design/基本設計書.md) |
| 【C】詳細設計 | 具体的にどう作るか（実装直前） | [詳細設計書](docs/detail-design/詳細設計書.md) |
| — | 技術選定の経緯 | [ADR (`docs/decisions/`)](docs/decisions/) |

[`docs/README.md`](docs/README.md) にも同様のインデックスがあります。

## 整備されている UML 図

| 図 | 場所 | 形式 |
|---|---|---|
| ユースケース図 | [`requirements/use-case/`](docs/requirements/use-case/) | Mermaid |
| アクティビティ図 | [`requirements/activity/`](docs/requirements/activity/) | Mermaid |
| コンポーネント図 | [`basic-design/architecture/`](docs/basic-design/architecture/) | Mermaid |
| クラス図（ドメインモデル） | [`basic-design/data-model/`](docs/basic-design/data-model/) | Mermaid + PlantUML |
| 画面遷移図 | [`basic-design/screen-flow/`](docs/basic-design/screen-flow/) | Mermaid |
| ER 図（論理） | [`basic-design/er/`](docs/basic-design/er/) | Mermaid |
| シーケンス図（SSO / 日記投稿 / 編集 ほか） | [`detail-design/sequence/`](docs/detail-design/sequence/) | Mermaid |
| 状態遷移図（セッション / 日記エントリ） | [`detail-design/state/`](docs/detail-design/state/) | Mermaid |
| OpenAPI 3.0 | [`detail-design/api/openapi.yaml`](docs/detail-design/api/openapi.yaml) | YAML |
| テーブル定義書（DDL 付き） | [`detail-design/table/tables.md`](docs/detail-design/table/tables.md) | Markdown |

## 図の見方

- **Mermaid**: GitHub 上でそのままレンダリングされます。Markdown プレビュー対応エディタ（VS Code / Obsidian 等）でも閲覧可。
- **PlantUML**: ローカルでレンダリングするには `plantuml` をインストールしてください。
  ```bash
  brew install plantuml
  plantuml -tsvg docs/basic-design/data-model/data-model.puml
  ```
  オンラインで開きたい場合は <https://www.plantuml.com/plantuml> にソースを貼り付けてください。

## 識別子の体系

ドキュメント横断のトレーサビリティのため、以下の ID 体系で参照しています。

- 機能ID: `FN-{ドメイン}-{連番}` 例: `FN-AUTH-01`, `FN-DIARY-03`
- 画面ID: `SC-{連番}` 例: `SC-02`
- API: `{METHOD} {path}` 例: `GET /api/diary-entries/{date}`
- テーブル: `snake_case` 例: `diary_entries`

## リポジトリ方針

- 生成物（ドキュメント・図・コメント）は **日本語** で記述
- 図はテキストベース（Mermaid / PlantUML）で管理し、git で差分を追える形にする
- 画像バイナリはコミットしない（必要に応じてローカルでレンダリング）
- 1ドキュメント1トピック、関連ドキュメントはリンクで参照

詳細は [`CLAUDE.md`](CLAUDE.md) を参照（Claude Code 向けの作業指示も兼ねています）。

## ステータス

| フェーズ | 状態 |
|---|---|
| 要件定義 | 初版完了 |
| 基本設計 | 初版完了 |
| 詳細設計 | 初版完了（API・DB・シーケンス・状態遷移まで） |
| 実装 | 未着手（本リポジトリのスコープ外） |
