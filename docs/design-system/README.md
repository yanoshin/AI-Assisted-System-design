# Design System — 画面UI/UX 設計

Daily Diary の **画面UI/UX 設計** を集約するパッケージです。要件・基本・詳細の3階層とは独立した「画面の見た目・状態・操作感」の観点をここに集めています。

> 本パッケージのスコープは **画面設計中心（軽量）**。デザイントークン（色・タイポ等）・コンポーネント仕様・UX リサーチは現時点では含めません。将来チームが拡張するためのプレースホルダとして残しています。

## ファイル構成

| ファイル / フォルダ | 内容 |
|---|---|
| [`information-architecture.md`](information-architecture.md) | サイトマップ・URL 設計・ナビゲーション体系 |
| [`interaction-states.md`](interaction-states.md) | 全画面共通の Loading / Empty / Error / Success の表現規約 |
| [`screens/`](screens/) | 画面別の詳細仕様（SC-ID ごとに1ファイル） |
| [`screens/_requests/`](screens/_requests/) | 指示者からの画面イメージ受け取り場所（ワイヤ・希望・参考） |

## 他の設計階層との関係

| 既存ドキュメント | このパッケージでの扱い |
|---|---|
| [要件定義書 §5](../requirements/要件定義書.md) | 機能ID と画面の対応元。`screens/sc-XX-*.md` から参照 |
| [基本設計書 §2](../basic-design/基本設計書.md) | 画面一覧・遷移方針の上位文書。詳細は本パッケージへ誘導 |
| [詳細設計書 §2](../detail-design/詳細設計書.md) | 画面項目・イベント仕様の技術視点。本パッケージは UI 視点で補完 |
| [画面遷移図](../basic-design/screen-flow/) | 画面間のフローはこちら。本パッケージは1画面の中身を扱う |

## 画面別 md のテンプレート

`screens/sc-XX-*.md` は以下のセクションを持つ。SC-03（[`screens/sc-03-compose.md`](screens/sc-03-compose.md)）が完全な記入例です。

1. **概要** — 目的 / 利用者 / 前提条件 / 関連機能ID・API
2. **ワイヤーフレーム** — ASCII または Mermaid `flowchart` でローファイ表現
3. **画面項目** — 名前 / 型 / 必須 / 桁 / 初期値 / 制約のテーブル
4. **イベント仕様** — トリガー / 動作 / 遷移先 のテーブル
5. **状態（States）** — Loading / Empty / Error / Success ごとの挙動
6. **アクセシビリティ** — 見出し階層・キーボード操作・フォーカス順序
7. **関連ドキュメント** — 要件・詳細設計・シーケンス図への相互リンク

## 指示者から画面イメージを伝える方法

[`screens/_requests/README.md`](screens/_requests/README.md) を参照。会話に ASCII ワイヤを貼る / `_requests/` に書く / ローカル画像を Read で渡す の3方式を併用します。
