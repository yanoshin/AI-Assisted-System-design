# ADR 0001: アーキテクチャ — 3層 + 外部 IdP

- **Status**: Accepted
- **Date**: 2026-05-25

## Context

Daily Diary は MVP で「会員登録 / SSO ログイン / 日記投稿・編集」を提供する。技術選定の自由度は高いが、以下の制約がある。

- 開発者数は少人数（MVP）
- リリースまでの期間を短くしたい
- 認証は SSO 専用、IdP は外部
- データは利用者数 10 万人 × 3 年分の規模に耐える必要

## Decision

`[Browser] → [CDN] → [Web (SSR)] → [API Server] → [PostgreSQL]` の 3 層構成 + 外部 Google OIDC を採用する。

- **Web (SSR)**: Next.js による SSR + クライアントサイドハイドレーション
- **API Server**: REST API。Web から HTTP で叩く
- **DB**: PostgreSQL 単一インスタンス（リードレプリカは将来）
- **IdP**: Google OIDC のみ

詳細は [基本設計書 §1](../basic-design/基本設計書.md#1-システム方式設計アーキテクチャ) を参照。

## Consequences

### Good
- 一般的な構成で学習コスト低、知見が豊富
- Web と API を分離しておくと、後でモバイルアプリ等を追加しやすい
- PostgreSQL は UNIQUE 制約・FK・トランザクションが揃っており、1日1エントリ制約（[ADR 0003](0003-one-entry-per-day-rule.md)）を DB レベルで保証できる

### Bad / Trade-offs
- Web と API を分離すると、ローカル開発時に2プロセス必要
- 初期は規模に対してオーバースペックの可能性。ただし将来の拡張余地は確保

## Alternatives

- **モノリス（Web/API 同居）**: 開発スピードは速いが、後の分離コストが高い
- **サーバレス (Functions)**: スケールは楽だが、長時間接続・コールドスタートが UX に影響
- **NoSQL (Firestore 等)**: UNIQUE 制約のような整合性保証が弱く、1日1エントリ制約の実装が複雑化
