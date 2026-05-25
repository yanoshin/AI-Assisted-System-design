# ユースケース図

Daily Diary の機能要件（誰が何をできるか）を表現するユースケース図。

- ソースファイル: [`use-case.mmd`](use-case.mmd)

## アクター

| アクター | 説明 |
|---|---|
| 一般利用者 (User) | 日記を書く / 読む人。アプリの主アクター |
| Google IdP | 外部認証プロバイダ。SSO ログインに利用 |
| 運用管理者 (Admin) | 障害対応・通報対応。MVP では UI なし（運用手順のみ） |

## ユースケース一覧

機能ID と1対1で対応する。詳細は [要件定義書 §5.2](../要件定義書.md#52-機能一覧) を参照。

| 機能ID | ユースケース |
|---|---|
| FN-AUTH-01 | SSO ログイン |
| FN-AUTH-02 | ログアウト |
| FN-AUTH-03 | 会員登録（初回ログイン時に自動） |
| FN-AUTH-04 | アカウント削除 |
| FN-DIARY-01 | 当日の日記投稿 |
| FN-DIARY-02 | 過去日記閲覧 |
| FN-DIARY-03 | 過去日記編集 |
| FN-DIARY-04 | 日記削除 |

## 図

```mermaid
flowchart LR
    User((一般利用者))
    Admin((運用管理者))
    Google[Google IdP]

    subgraph DailyDiary[Daily Diary]
        UC1([SSO ログイン])
        UC2([ログアウト])
        UC3([会員登録])
        UC4([アカウント削除])
        UC5([当日の日記投稿])
        UC6([過去日記閲覧])
        UC7([過去日記編集])
        UC8([日記削除])
    end

    User --- UC1
    User --- UC2
    User --- UC4
    User --- UC5
    User --- UC6
    User --- UC7
    User --- UC8

    UC1 -.include.-> UC3
    UC1 --- Google

    Admin -.運用窓口.- DailyDiary
```

> Mermaid のユースケース表現は `flowchart` で代用している（標準のユースケース図記法がないため）。`(())` をアクター、`([...])` をユースケースと読み替えてほしい。PlantUML 版が必要になれば併設する。
