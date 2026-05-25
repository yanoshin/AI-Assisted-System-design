# 技術スタック 採用記録

[`interview.md`](interview.md) の各質問への回答をここに記録します。**未回答の欄は空欄のまま** にしておき、決まったら更新してください。

## メタ情報

| 項目 | 値 |
|---|---|
| 対象プロジェクト | Daily Diary |
| 記入者 | （指示者 + Claude Code によるインタビュー） |
| 最終更新 | 2026-05-26 |
| 関連 ADR | [`../decisions/`](../decisions/) |

## 採用記録

| Q-ID | 質問要約 | 採用 | 採用理由 | 代替案・将来検討 | 決定日 |
|---|---|---|---|---|---|
| Q-PRJ-01 | プロジェクト規模・期間 | B. 本番運用・中期（1〜2年） | KPI（7日継続率30% / 月10投稿）と保守性を両立 | C への移行（長期運用化） | 2026-05-26 |
| Q-PRJ-02 | 既存スキル | A. 既存スキル優先 | MVP の早期達成を優先 | B（新技術を一部導入） | 2026-05-26 |
| Q-PRJ-03 | 予算・運用コスト | B. バランス | 利用者規模（数万人上限）と中期運用に妥当 | C（運用負荷最小化） | 2026-05-26 |
| Q-PRJ-04 | ロックイン許容度 | B. 特定領域は許容 | DB・ストレージなどはクラウド固有を許可 | A（避けたい） | 2026-05-26 |
| Q-FE-01 | レンダリング戦略 | A. SSR | ADR 0001 と整合、SEO・初期表示重視 | D（ISR/PPR で部分最適化） | 2026-05-26 |
| Q-FE-02 | UI フレームワーク | A. React (Next.js) | 基本設計書 §1.2 で採用、エコシステム最大 | B（Vue / Nuxt） | 2026-05-26 |
| Q-FE-03 | CSS 手法 | A. Tailwind CSS | 変更容易、Next.js と相性、Daily Diary のシンプル指向に合致 | B（CSS Modules） | 2026-05-26 |
| Q-FE-04 | 状態管理 | C+A. TanStack Query + React Context | サーバ状態と局所 UI 状態を分離、規模相応 | B（Zustand） | 2026-05-26 |
| Q-BE-01 | バックエンド言語 | A. TypeScript (Node.js) | フロントと言語統一、型・スキーマ共有 | C（Go） | 2026-05-26 |
| Q-BE-02 | API スタイル | A. REST | OpenAPI で既に REST 設計済み | D（tRPC） | 2026-05-26 |
| Q-BE-03 | バックエンド FW | A. Fastify | 高速・軽量、Cookie / OIDC ライブラリ互換性 | B（Hono、Edge SSR への展開時） | 2026-05-26 |
| Q-BE-04 | 非同期 / ジョブ | A. 不要 | BAT-01（セッション掃除）はクラウドスケジューラで対応 | B（BullMQ、機能拡張時） | 2026-05-26 |
| Q-DATA-01 | 主データストア | **B. PostgreSQL** | ADR 0001 で採用済み、DDL も PostgreSQL 16 前提。**GCP Cloud SQL for PostgreSQL を想定** | A（MySQL） | 2026-05-26 |
| Q-DATA-02 | NoSQL の種類 | **N/A** | NoSQL ではないため | — | 2026-05-26 |
| Q-DATA-03 | キャッシュ | A. 不要 | MVP は DB のみで十分。読取量が増えたら Redis 追加 | C（Redis / Memorystore） | 2026-05-26 |
| Q-DATA-04 | 全文検索 | A. 不要 | 全文検索は MVP 対象外（要件 §2.2） | B（PG tsvector） | 2026-05-26 |
| Q-AUTH-01 | 認証方式 | A. SSO (OIDC / OAuth) | 要件で確定。Google OIDC を ADR 0002 で選定済み | — | 2026-05-26 |
| Q-AUTH-02 | 認可モデル | A. シンプル所有者 | 詳細設計書の方針通り、`user_id == session.user_id` で十分 | B（RBAC、管理者機能拡張時） | 2026-05-26 |
| Q-OBS-01 | 監視3点セット | A. メトリクス + ログ | MVP 規模では十分。トレースは成熟期に追加 | B（OpenTelemetry 3点） | 2026-05-26 |
| Q-OBS-02 | エラートラッキング | A. 不要 | クラウド標準のログで対応 | B（Sentry、本番運用安定後に追加） | 2026-05-26 |
| Q-CI-01 | CI/CD | **D. クラウド標準（Cloud Build）** | GCP 採用に合わせる | A（GitHub Actions と併用も可） | 2026-05-26 |
| Q-CI-02 | テスト方針 | B. 単体 + 結合 + E2E | 認証フロー（OIDC）は結合テスト必須 | A（単体 + E2E に縮小） | 2026-05-26 |

## 採用一覧（サマリ）

```
- フロントエンド: Next.js (React, SSR) + Tailwind CSS + TanStack Query + React Context
- バックエンド:   Node.js (TypeScript) + Fastify, REST API
- データベース:   GCP Cloud SQL for PostgreSQL (PostgreSQL 16)
- 認証:           Google OIDC (SSO) + シンプル所有者ベース認可
- 監視:           Cloud Monitoring + Cloud Logging（クラウド標準）
- CI/CD:          Google Cloud Build
- テスト:         単体 (Vitest) + 結合 (testcontainers) + E2E (Playwright)
- ロックイン方針: DB / 監視 / CI など特定領域は GCP 固有サービスを許容
```

## 大きな決定は ADR へ昇格

技術選定のうち「**変更時の影響が大きい**もの」「**今後の判断を縛るもの**」は、本ファイルに加えて [`../decisions/`](../decisions/) に ADR として独立記録する。例:
- データベース選定（移行コストが大きい）
- 認証方式の選定（セキュリティ影響が大きい）
- IaC ツールの選定（運用全体に波及する）

### Daily Diary で既に ADR 化済み
- [ADR 0001 アーキテクチャ](../decisions/0001-architecture.md) — 3層 + 外部 IdP
- [ADR 0002 SSO プロバイダの選定](../decisions/0002-sso-provider-selection.md) — Google OIDC のみ
- [ADR 0003 1日1エントリ制約](../decisions/0003-one-entry-per-day-rule.md)

### 今回のインタビューで ADR 化を検討すべき決定
- **クラウド選定 (GCP)**: 後続のインフラインタビューで確定後、ADR 0004 として独立記録を推奨
- **CI/CD 選定 (Cloud Build)**: 上記と同じ流れで ADR 化検討
