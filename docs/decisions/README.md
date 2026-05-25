# 意思決定記録 (ADR)

設計上の主要な意思決定とその根拠を時系列で記録する。形式は MADR ライク（Markdown Architectural Decision Records）。

## 一覧

| ID | タイトル | ステータス |
|---|---|---|
| [0001](0001-architecture.md) | アーキテクチャ: 3層 + 外部 IdP | Accepted |
| [0002](0002-sso-provider-selection.md) | SSO プロバイダの選定 | Accepted |
| [0003](0003-one-entry-per-day-rule.md) | 1日1エントリ制約の採用 | Accepted |
| [0004](0004-cloud-platform-selection.md) | クラウド選定 — GCP | Accepted |
| [0005](0005-waf-cdn-deferred.md) | WAF / CDN を MVP では不採用 | Accepted |

## 書き方の指針

各 ADR は以下のセクションを持つ:
- **Status**: Proposed / Accepted / Deprecated / Superseded by ...
- **Context**: なぜ意思決定が必要だったか
- **Decision**: 何を決めたか
- **Consequences**: 良い結果と悪い結果、トレードオフ
- **Alternatives**: 検討した別案
