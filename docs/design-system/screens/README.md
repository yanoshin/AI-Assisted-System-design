# 画面別仕様

Daily Diary の各画面の UI 設計。1画面1ファイルで、SC-ID と機能ID（要件定義 §5.2）にトレース可能。

## 画面一覧

| SC-ID | 画面 | ファイル | 認証 | 関連機能ID |
|---|---|---|---|---|
| SC-01 | ランディング / ログイン | [sc-01-landing.md](sc-01-landing.md) | 不要 | FN-AUTH-01, FN-AUTH-03 |
| SC-02 | ダッシュボード | [sc-02-dashboard.md](sc-02-dashboard.md) | 必要 | — |
| SC-03 | 日記投稿 | [sc-03-compose.md](sc-03-compose.md) | 必要 | FN-DIARY-01 |
| SC-04 | 日記一覧 / カレンダー | [sc-04-list.md](sc-04-list.md) | 必要 | FN-DIARY-02 |
| SC-05 | 日記詳細・編集 | [sc-05-detail-edit.md](sc-05-detail-edit.md) | 必要 | FN-DIARY-02, FN-DIARY-03, FN-DIARY-04 |
| SC-06 | アカウント設定 | [sc-06-account.md](sc-06-account.md) | 必要 | FN-AUTH-02, FN-AUTH-04 |

## テンプレート

各画面 md は以下の見出し構成で書く。完全な記入例は **[sc-03-compose.md](sc-03-compose.md)** を参照。

```
# SC-XX: 画面名
## 概要         （目的・利用者・前提・関連機能ID/API/シーケンス）
## ワイヤーフレーム  （ASCII or Mermaid）
## 画面項目      （項目テーブル）
## イベント仕様    （トリガー / 動作 / 遷移先）
## 状態         （Loading / Empty / Error / Success）
## アクセシビリティ  （見出し・キーボード・SR）
## 関連ドキュメント   （要件・詳細・シーケンス）
```

状態の表現規約は [`../interaction-states.md`](../interaction-states.md)、サイト全体の構造は [`../information-architecture.md`](../information-architecture.md) を参照。

## 指示者から画面イメージを伝える

[`_requests/`](_requests/) に書いてもらえれば、対応する `sc-XX-*.md` に反映します。詳細は [`_requests/README.md`](_requests/README.md)。
