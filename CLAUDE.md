# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリについて

AIシステムの**設計ドキュメント集**。アプリケーションコードではなく、設計書とUML図を管理する。実装の技術スタックは未定で、ドキュメントとモデリングが中心。

## 方針

- 生成物（ドキュメント・図・コメント・回答）は**日本語**で書く。
- ドキュメントはMarkdown。UMLはテキストベースで記述し、gitで差分を追える形にする。GitHub等でそのまま描画される **Mermaid** を基本とし、より高度な記法が必要な場合は **PlantUML** を使う。1つのドキュメント内では記法を統一する。
- 図は画像バイナリではなくソーステキスト（`.md` / `.puml` / `.mmd`）で管理し、必要に応じてローカルでレンダリングする。
- 1ドキュメント1トピック。関連ドキュメントは内容を重複させずにリンクする。

## 構成の目安

内容が増えてきたら `docs/` 配下を関心ごとに整理する（例: `docs/architecture/`, `docs/uml/`, 意思決定記録は `docs/decisions/`）。これは出発点であり固定ルールではない。題材に合わせて調整する。

## ワークフロー

- ユーザーが「**現状確認してください**」（およびそれに類する短い指示）と言った場合は、「現状確認 + 必要に応じて各種mdファイルの生成・更新」までを一連の作業として扱う。具体的には次の手順で進める。
  1. `docs/` 配下のファイル一覧と各ドキュメントの現状（見出しのみのスケルトンか、本文まで埋まっているか、UML図の有無）を把握する。
  2. 不足や未着手の部分を洗い出し、要約として報告する。
  3. 続けて埋めるべき内容があれば、方針を提示してから（必要に応じて確認を取り）mdファイルを生成・更新する。

### 技術スタック / インフラ設計のインタビュー進行

指示者が以下のいずれかを依頼した場合、Claude は対応する `interview.md` を読み、質問を順に `AskUserQuestion` で確認しながら、回答を `decision-record.md` / `design.md` / `architecture/*.mmd` に逐次反映する。

- 「**技術スタックを決めたい**」「**tech-stack のインタビューを進めて**」など → [`docs/tech-stack/interview.md`](docs/tech-stack/interview.md)
- 「**インフラ設計を決めたい**」「**infrastructure のインタビューを進めて**」など → [`docs/infrastructure/interview.md`](docs/infrastructure/interview.md)

進め方:
1. 該当の `interview.md` を読んで質問リストを把握
2. 1〜数問ずつ `AskUserQuestion` を投げて回答を取得（4問/1メッセージが上限）
3. 回答が出るたびに `decision-record.md` / `design.md` に反映
4. インフラ設計でクラウド選定が決まったら、[`service-mapping.md`](docs/infrastructure/service-mapping.md) を参照して具体サービス名に展開
5. 必要に応じて `architecture/*-template.mmd` をコピーして実構成図を起こす

### 画面UIイメージの伝達手段

指示者が画面UIのイメージを伝えるときは、以下のいずれか（または併用）を使う。Claude は受け取ったイメージを、対応する `docs/design-system/screens/sc-XX-*.md` に反映する。

1. **会話に ASCII ワイヤを貼る** — 軽い要望はその場で罫線で示す
2. **`docs/design-system/screens/_requests/sc-XX.md` に書く** — 議論履歴を git に残したいとき。テンプレは [`_requests/README.md`](docs/design-system/screens/_requests/README.md)
3. **ローカルパスの画像を Read で渡す** — 既存ラフ・スクショ・手書きスケッチを共有したいとき。例: 「`~/Desktop/wireframe.png` を Read で見て」

画像ファイルはリポジトリにコミットしない（テキスト・ベクターのみという既存方針を維持）。リポジトリ外（`/tmp/` や `~/Desktop/` など）に置いてパスだけ共有する。

## メモ

- ビルド/lint/テストのツールは存在しない（ドキュメント専用リポジトリ）。後でアプリケーションコードを追加した場合は `/init` を再実行する。
