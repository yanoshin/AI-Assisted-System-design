# 業務フロー（アクティビティ図）

利用者から見た主要シナリオの流れ。

| ファイル | シナリオ | 対応シナリオID |
|---|---|---|
| [`signup-and-first-post.mmd`](signup-and-first-post.mmd) | 初回登録〜初回投稿 | SC-USE-01 |
| [`edit-past-entry.mmd`](edit-past-entry.mmd) | 過去日記の振り返り編集 | SC-USE-03 |

## 初回登録〜初回投稿

```mermaid
flowchart TD
    Start([開始]) --> Land[ランディング画面 SC-01 にアクセス]
    Land --> ClickLogin{Google でログインを押す}
    ClickLogin --> Google[Google IdP で認証・同意]
    Google -->|同意| Callback[コールバック受領 / ID Token 検証]
    Google -->|拒否| Land
    Callback --> Exists{User 登録済?}
    Exists -->|No| Create[User 自動作成 FN-AUTH-03]
    Exists -->|Yes| Session
    Create --> Session[セッション発行]
    Session --> Dash[ダッシュボード SC-02 表示]
    Dash --> Today{当日のエントリ有?}
    Today -->|No| Compose[投稿画面 SC-03 へ]
    Today -->|Yes| EditExisting[編集画面 SC-05 へ]
    Compose --> Input[タイトル・本文を入力]
    Input --> Save[保存ボタン押下]
    Save --> Persist[エントリ作成 FN-DIARY-01]
    Persist --> Done([完了])
    EditExisting --> Done
```

## 過去日記の振り返り編集

```mermaid
flowchart TD
    Start([開始]) --> Dash[ダッシュボード SC-02]
    Dash --> List[一覧 / カレンダー SC-04 を開く]
    List --> Select[対象日のエントリを選択]
    Select --> Detail[詳細・編集画面 SC-05 表示]
    Detail --> Action{何をする?}
    Action -->|閲覧のみ| Done([完了])
    Action -->|編集| Edit[タイトル・本文を変更]
    Action -->|削除| Confirm{削除確認?}
    Edit --> Update[更新ボタン押下 FN-DIARY-03]
    Update --> Saved[更新成功表示] --> Done
    Confirm -->|Yes| Delete[物理削除 FN-DIARY-04] --> List
    Confirm -->|No| Detail
```
