# シーケンス図

機能ごとの処理フロー。すべて Mermaid `sequenceDiagram` 形式。

| ファイル | 機能ID | 対応シナリオ |
|---|---|---|
| [`sso-login.mmd`](sso-login.mmd) | FN-AUTH-01 | SSO ログイン（OIDC Authorization Code Flow + PKCE） |
| [`signup-email.mmd`](signup-email.mmd) | FN-AUTH-03 | 初回 SSO 時の自動アカウント作成（※ファイル名は歴史的経緯） |
| [`logout.mmd`](logout.mmd) | FN-AUTH-02 | ログアウト |
| [`post-diary.mmd`](post-diary.mmd) | FN-DIARY-01 | 当日の日記投稿 |
| [`edit-past-diary.mmd`](edit-past-diary.mmd) | FN-DIARY-03 | 過去日記の編集 |

## 共通参加者

| 表記 | 実体 |
|---|---|
| 利用者 | 一般利用者（ペルソナ: 平岡 はるか） |
| ブラウザ | クライアント |
| API Server | Fastify / Hono 想定 |
| PostgreSQL | プライマリ DB |
| Google OIDC | 外部 IdP |
