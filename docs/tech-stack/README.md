# 技術スタック決定パッケージ

開発する技術スタック（言語・フレームワーク・ライブラリ・運用ツール）を **インタビュー形式** で決めるための雛形です。

## 進め方

```
1. interview.md       … 質問集をざっと眺める
        ↓
2. AskUserQuestion / 自己回答  … 各質問に答える
        ↓
3. decision-record.md  … 採用と理由を記入
        ↓
4. catalog.md         … 候補一覧で詳細を確認・補強
```

### 自走する場合
- `interview.md` を上から順に読んで `decision-record.md` の表に書き込んでいく

### Claude にインタビューを進めてもらう場合
- 「**tech-stack のインタビューを進めて**」と依頼
- Claude は `interview.md` を読んで、1〜数問ずつ `AskUserQuestion` を投げ、回答を `decision-record.md` に逐次反映
- ワークフロー詳細は [CLAUDE.md](../../CLAUDE.md) を参照

## ファイル構成

| ファイル | 役割 |
|---|---|
| [`interview.md`](interview.md) | 質問集（プロジェクト前提 / フロント / バック / データ / 認証 / 監視 / CI-CD） |
| [`decision-record.md`](decision-record.md) | 採用結果の記入欄（Q-ID × 採用 × 理由 × 代替案） |
| [`catalog.md`](catalog.md) | レイヤー別の代表的候補と Pros/Cons の参照集 |

## 既存ドキュメントとの関係

| 既存 | このパッケージでの扱い |
|---|---|
| [基本設計書 §1.2 技術スタック](../basic-design/基本設計書.md#12-技術スタック推奨) | 採用済みスタックの「結論」のみ。**決定プロセス** はこちら |
| [ADR](../decisions/) | 大粒度の方針決定。技術選定は ADR に昇格させてもよい |

## 関連: インフラ設計

サーバ・ネットワーク・IaC の整備は [`../infrastructure/`](../infrastructure/) で扱う。技術スタック側はアプリ実装の選定、インフラ側は実行環境の選定と分担する。
