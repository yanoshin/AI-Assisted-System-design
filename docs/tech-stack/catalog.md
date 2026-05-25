# 技術スタック 候補カタログ

レイヤー別の代表的な選択肢と Pros / Cons。`interview.md` の各質問から参照される参考集。

---

## フロントエンド

### レンダリング戦略
| 戦略 | Pros | Cons | 主な実装 |
|---|---|---|---|
| SSR | SEO に強い、初期表示が速い | サーバ負荷、運用が複雑 | Next.js / Nuxt / SvelteKit / Remix |
| SPA | 単純、CDN だけで配信可 | 初期表示遅い、SEO 弱い | React / Vue / Svelte + Vite |
| SSG | 高速・低コスト | 動的データに不向き | Astro / Next.js (export) / Hugo |
| ISR / PPR | SSR と SSG のいいとこ取り | フレームワーク依存・難解 | Next.js (ISR/PPR) |

### UI フレームワーク
| FW | Pros | Cons |
|---|---|---|
| React | エコシステム最大、求人多 | 自由度高すぎて選択肢過多 |
| Vue | 学習曲線がゆるい、文書が親切 | エコシステムは React より小 |
| Svelte | 軽量・コンパイル時最適化 | 求人・知見は相対的に少ない |
| Solid / Qwik | 性能高い | 採用例少 |

### CSS
| 手法 | Pros | Cons |
|---|---|---|
| Tailwind CSS | クラス命名不要、変更容易 | HTML が肥大化、学習曲線あり |
| CSS Modules | スコープ付き、シンプル | コンポーネント単位で分散 |
| CSS-in-JS | 動的 CSS が容易 | ランタイムコスト |
| Vanilla CSS | 依存ゼロ | スケールしにくい |

### 状態管理
| 手法 | 用途 |
|---|---|
| TanStack Query / SWR | サーバ状態（API キャッシュ） |
| Zustand / Jotai | 軽量なグローバル状態 |
| Redux Toolkit | 大規模・歴史あるアプリ |
| React Context / Pinia | フレームワーク標準で十分な規模 |

---

## バックエンド

### 言語
| 言語 | Pros | Cons | 適合 |
|---|---|---|---|
| TypeScript (Node.js) | フロントと統一、エコシステム広い | 性能は最高ではない | Web API 全般 |
| Python | データ・ML との親和性 | 性能・並列性 | データ系、AI 連携 |
| Go | 高速・シンプル・並列性 | 抽象度低い | 高負荷 API、CLI |
| Rust | 性能・安全性 | 学習曲線急 | 性能クリティカル |
| Java / Kotlin | 安定・大規模 | 起動遅い・記述量 | エンタープライズ |
| C# | エコシステム成熟 | Windows 文化 | Azure / Unity 関連 |

### TypeScript 系 Web FW
| FW | 特徴 |
|---|---|
| Fastify | 高速・軽量、プラグイン豊富 |
| Hono | Edge / Workers にも対応、超軽量 |
| NestJS | DI / モジュール、大規模向け |
| Express | 知名度・知見豊富、古典的 |
| tRPC | TS 統一なら型共有が強力 |

### API スタイル
| スタイル | 適合 |
|---|---|
| REST | 標準的・キャッシュしやすい・学習低 |
| GraphQL | クライアントが多様・柔軟クエリ |
| gRPC | マイクロサービス間・性能重視 |
| tRPC | TS フルスタック・小〜中規模 |

### ジョブ / メッセージング
| 手段 | 用途 |
|---|---|
| BullMQ / pg-boss | 軽量ジョブキュー（Redis / Postgres） |
| Sidekiq (Ruby) | Ruby 系 |
| Kafka | 大規模ストリーミング |
| NATS / RabbitMQ | メッセージブローカ |
| SQS / Pub/Sub | クラウドマネージド |

---

## データ層

### DBMS
| DB | Pros | Cons |
|---|---|---|
| PostgreSQL | 機能豊富、JSON 扱える、知見広い | チューニングがやや難 |
| MySQL / MariaDB | 普及度・運用知見 | 機能は Postgres より控えめ |
| SQLite | 単一プロセス・軽量・本番でも可 | 同時書込弱い |
| Cloud Spanner / CockroachDB | グローバル分散 | 高コスト・複雑 |

### NoSQL
| DB | 用途 |
|---|---|
| MongoDB | ドキュメント、柔軟スキーマ |
| DynamoDB / Firestore | サーバレス、高スケール |
| Cassandra / ScyllaDB | 書き込み重視・大規模 |

### キャッシュ / 検索
| ツール | 用途 |
|---|---|
| Redis | キャッシュ・セッション・軽量キュー |
| Memcached | 純粋キャッシュ |
| Meilisearch / Typesense | OSS 全文検索（軽量） |
| OpenSearch / Elasticsearch | 大規模全文検索・ログ集約 |
| Algolia | SaaS 検索（高速・低運用） |

---

## 認証・認可

| 手段 | 適合 |
|---|---|
| Google / GitHub / Apple OIDC | SSO 専用、ID 連携 |
| Auth0 / Firebase Auth / Clerk | SaaS、UI まで提供 |
| Cognito / Identity Platform | クラウド標準 |
| Keycloak | OSS、自前ホスト |
| 自前 (bcrypt + JWT/Session) | コントロール最大、責任重 |

| モデル | 適合 |
|---|---|
| 所有者ベース | 1利用者が自分のリソースのみ操作 |
| RBAC | ロールが少数（管理者・利用者） |
| ABAC | 属性で動的に判定 |
| ReBAC | リソース間の関係を表現（Google Drive 型） |

---

## 監視・ロギング

| ツール | 用途 |
|---|---|
| OpenTelemetry | Metrics + Logs + Traces の標準 |
| Prometheus + Grafana | OSS メトリクス |
| Loki | OSS ログ集約 |
| Datadog / New Relic | SaaS フルスタック |
| Sentry | エラートラッキング定番 |
| CloudWatch / Cloud Monitoring | クラウド標準 |

---

## CI/CD・テスト

| ツール | 用途 |
|---|---|
| GitHub Actions | GitHub 完結・OSS フレンドリ |
| GitLab CI | GitLab 完結 |
| CircleCI / Buildkite | SaaS、並列性 |
| ArgoCD / Flux | GitOps（K8s） |

| テスト | ツール |
|---|---|
| 単体 | Vitest / Jest / Pytest / Go test |
| 結合 | testcontainers / supertest |
| E2E | Playwright / Cypress |
| 性能 | k6 / Locust / Gatling |
| セキュリティ | OWASP ZAP / Trivy / Snyk |
