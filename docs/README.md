# 設計ドキュメント一覧

題材: **Daily Diary**（毎日の日記を投稿・編集できる SSO 対応 Web アプリケーション）

設計工程の流れに沿って **要件定義 → 基本設計 → 詳細設計** の3階層で構成しています。

## ドキュメント階層

| 階層 | 答えるべき問い | エントリ |
|---|---|---|
| 【A】要件定義 | **何を**作るか / なぜ作るか | [要件定義書](requirements/要件定義書.md) |
| 【B】基本設計 | **どう**作るか（全体像・方式） | [基本設計書](basic-design/基本設計書.md) |
| 【C】詳細設計 | **具体的に**どう作るか（実装直前の粒度） | [詳細設計書](detail-design/詳細設計書.md) |

加えて、技術選定の経緯は [ADR (`decisions/`)](decisions/) に、画面UI/UX 設計のサンプル一式は [Design System (`design-system/`)](design-system/) に整備しています。

## UML の使い分け（方針）

| UML / 図 | 主な用途 | 整備場所 |
|---|---|---|
| ユースケース図 | 機能要件（誰が何をできるか） | [requirements/use-case/](requirements/use-case/) |
| アクティビティ図 | 業務フロー / 利用シナリオ | [requirements/activity/](requirements/activity/) |
| コンポーネント図 / 配置図 | アーキテクチャ・インフラ | [basic-design/architecture/](basic-design/architecture/) |
| クラス図 | ドメインモデル | [basic-design/data-model/](basic-design/data-model/) |
| 画面遷移図 | 画面の遷移 | [basic-design/screen-flow/](basic-design/screen-flow/) |
| ER図 | データベース構造（論理） | [basic-design/er/](basic-design/er/) |
| シーケンス図 | 機能の処理フロー | [detail-design/sequence/](detail-design/sequence/) |
| 状態遷移図 | ライフサイクル（セッション・日記） | [detail-design/state/](detail-design/state/) |

図は **Mermaid** を基本とし、必要に応じて **PlantUML** を併記する（[CLAUDE.md](../CLAUDE.md) の方針に従う）。

## 識別子の体系

ドキュメント横断のトレーサビリティのため、以下の ID 体系で参照する。

- 機能ID: `FN-{ドメイン}-{連番}` 例: `FN-AUTH-01`, `FN-DIARY-03`
- 画面ID: `SC-{連番}` 例: `SC-02`
- API: `{METHOD} {path}` 例: `GET /api/diary-entries/{date}`
- テーブル: `snake_case` 例: `diary_entries`

## MVP スコープ（要点）

詳細は要件定義書を参照。本リポジトリは MVP 範囲を前提に設計を起こしている。

- 認証は **SSO 専用**（対応 IdP は **Google** の1社、OIDC Authorization Code Flow + PKCE）
- **1日1エントリ**制約あり（`(user_id, entry_date)` UNIQUE）
- 過去日記は **無期限で編集可能**（編集履歴は MVP では保持しない、`updatedAt` のみ）
- 日記は **プライベートのみ**（公開機能なし）
- 本文は **テキストのみ**（画像添付は MVP 対象外）
