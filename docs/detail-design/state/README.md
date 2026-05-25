# 状態遷移図

| ファイル | 対象 |
|---|---|
| [`session-state.mmd`](session-state.mmd) | `Session` のライフサイクル（Active / Expired / Revoked） |
| [`diary-entry-state.mmd`](diary-entry-state.mmd) | `DiaryEntry` のライフサイクル（Created / Edited / Deleted） |

## 補足

MVP では日記の「下書き」「公開」「非公開」のような状態区分は持たない。`DiaryEntry` は **存在する** か **物理削除済み** かの二択で、編集の有無は `updated_at` の差で表現する。

`Session` は DB レコードの `expires_at` / `revoked_at` から状態を派生させる（カラムとしては持たない）。BAT-01 が古い失効レコードを物理削除する。
