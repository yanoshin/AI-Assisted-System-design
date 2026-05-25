# ADR 0004: クラウド選定 — GCP (Google Cloud)

- **Status**: Accepted
- **Date**: 2026-05-26
- **関連**: [ADR 0001 アーキテクチャ](0001-architecture.md), [tech-stack/decision-record.md](../tech-stack/decision-record.md), [infrastructure/design.md](../infrastructure/design.md)

## Context

[ADR 0001](0001-architecture.md) は `[Browser] → [CDN] → [Web/SSR] → [API] → [PostgreSQL]` の3層構成を **クラウド非依存** で定義した。実際にホスティングするクラウドは未確定だった。

infrastructure インタビュー（[`infrastructure/interview.md`](../infrastructure/interview.md)）の Q-HOST-01 / Q-HOST-02 でホスティング先を選定する必要が生じた。判断材料:

- 既存決定: PostgreSQL（ADR 0001）、Google OIDC（[ADR 0002](0002-sso-provider-selection.md)）
- tech-stack 側の制約: Q-PRJ-04 で「特定領域はクラウド固有を許容」、Q-CI-01 で CI/CD はクラウド標準（GCP なら Cloud Build）
- 要件定義書 §2.3: データ保管リージョンは日本国内
- チームの既存スキル（Q-PRJ-02 = 既存スキル優先）
- MVP コスト（Q-PRJ-03 = バランス）
- マネージドサービス志向（Q-COMP-01 = コンテナマネージド）

## Decision

**GCP (Google Cloud) を採用** する。具体的な採用サービスは [infrastructure/design.md](../infrastructure/design.md) のとおり。要点:

| レイヤ | GCP サービス |
|---|---|
| コンピュート | **Cloud Run** (Web + API 同居) |
| データベース | **Cloud SQL for PostgreSQL 16** (Private IP) |
| ネットワーク | VPC + Cloud Load Balancing |
| シークレット | Secret Manager |
| 監視・ログ | Cloud Monitoring + Cloud Logging |
| CI/CD | Cloud Build + Artifact Registry |
| バッチ | Cloud Scheduler + Cloud Run jobs |
| リージョン | **asia-northeast1（東京）単一** |

## Consequences

### Good
- **Cloud Run のサーバレス特性**: `min=0` でアイドル時コストゼロ、`max` 設定で上限制御。MVP のコスト要件にマッチ
- **マネージド PostgreSQL**: バックアップ・パッチ適用が自動、運用負荷低
- **国内リージョン**で個人情報保護法の要件と整合
- **Cloud SQL と Cloud Run の親和性** (Serverless VPC Access で Private IP 接続)
- 単一ベンダーで完結することによる学習・運用コスト最小化

### Bad / Trade-offs
- **ベンダーロックイン**が増える（Q-PRJ-04 で「特定領域は許容」を選択済みなのでこの範囲）
- GCP 障害がそのままサービス停止に直結（緩和: 単一ゾーン障害には Cloud Run のリージョン分散で耐性あり、リージョン障害は対象外）
- マルチクラウド化の選択肢は事実上なくなる
- 将来 AWS への移行が必要になった場合、再設計コスト大

## Alternatives

| 案 | Pros | Cons | 不採用理由 |
|---|---|---|---|
| **AWS** | シェア最大、サービス最多、求人多 | Cloud Run 相当 (App Runner / Fargate) の運用設定が GCP より多い | GCP Cloud Run のシンプルさを優先 |
| **Azure** | Microsoft 製品との親和性、エンタープライズ | 今回 Microsoft 関連の要件なし | 動機なし |
| **マルチクラウド** | リスク分散 | コスト・複雑さの増加、運用負荷大 | MVP では過剰 |
| **Cloudflare / Vercel 等の PaaS** | 開発体験良好、Edge SSR が容易 | DB 周りの選択肢が狭い、ロックインがより強い | DB（Cloud SQL）との組み合わせで GCP が自然 |
| **オンプレ** | 究極のコントロール | 初期投資・運用ノウハウ要 | スタートアップ MVP には不適合 |

## 将来の見直しトリガー

以下のいずれかが発生した場合、ADR で再評価する:

- DAU 1万人超え時のコスト試算で他クラウドが大幅に有利と判明した場合
- GCP が特定サービス（Cloud Run / Cloud SQL）を廃止 or 大幅値上げした場合
- 法的・契約上の理由で別クラウドへの移行要求が発生した場合
- リージョン障害が連続して発生し、SLO を満たせない実績が出た場合

## 関連する追加検討事項

- **環境分離方針**: Q-ENV-02 で「同一プロジェクト内で命名プレフィックスで分離」を選択。利用者数増・コンプラ強化のタイミングで **GCP プロジェクト分離への昇格** を ADR 化候補（[infrastructure/design.md §11](../infrastructure/design.md) 参照）
- **WAF/CDN 不採用**: [ADR 0005](0005-waf-cdn-deferred.md) を参照
