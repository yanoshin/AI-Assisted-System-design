# ADR 0002: SSO プロバイダの選定

- **Status**: Accepted
- **Date**: 2026-05-25

## Context

要件で「会員登録 + ログイン・ログアウト（シングルサインオン）」が指定された。SSO を採用するにあたり、対応する IdP（Identity Provider）を選ぶ必要がある。

候補:
- Google (OIDC)
- GitHub (OAuth 2.0)
- Apple Sign in with Apple
- Microsoft (OIDC)
- 自前の SAML IdP

## Decision

**MVP では Google OIDC のみを採用** する。

- プロトコル: OIDC Authorization Code Flow + PKCE
- ID Token 検証は `iss` / `aud` / `exp` / `nonce` / 署名すべて実施
- 連携情報は `sso_identities (provider, provider_subject)` で管理し、将来プロバイダを増やせる設計

詳細は [詳細設計書 §7.3](../detail-design/詳細設計書.md#73-ssooidc実装詳細) を参照。

## Consequences

### Good
- 国内利用者のカバレッジが高い（Google アカウント保有率）
- OIDC は標準化されていてライブラリも豊富（`openid-client` 等）
- 1社に絞ることで実装・運用・テストのコストを最小化

### Bad / Trade-offs
- Google アカウントを持たない利用者を取りこぼす
- Google 側の障害がそのままサービス停止に直結する（緩和: ログイン後のセッションは30日有効）

## Alternatives

- **GitHub も追加**: エンジニア層には刺さるが、一般ペルソナ（[要件定義書 §3.3](../requirements/要件定義書.md#33-ペルソナ)）には不要
- **メール + パスワード を併存**: パスワード管理の責任を負うことになり、要件「シングルサインオン」とも整合しないので除外
- **Sign in with Apple**: iOS 配信時の必須要件になるが、MVP は Web のみのため不要

## 将来の見直しトリガー

- DAU が 5,000 を超え、登録時の離脱率が「Google アカウントを持っていない」理由で 5% を超えた段階で再評価
