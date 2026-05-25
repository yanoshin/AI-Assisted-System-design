# ADR 0005: WAF / CDN を MVP では不採用

- **Status**: Accepted
- **Date**: 2026-05-26
- **関連**: [ADR 0004 クラウド選定](0004-cloud-platform-selection.md), [基本設計書 §8.3](../basic-design/基本設計書.md), [詳細設計書 §8.2](../detail-design/詳細設計書.md), [infrastructure/design.md §3](../infrastructure/design.md)

## Context

既存設計書では以下のとおり WAF / CDN を **推奨** していた:

- [基本設計書 §8.3 脅威モデリング](../basic-design/基本設計書.md): 「Denial of Service → レートリミット、CDN」
- [詳細設計書 §8.2 ネットワーク詳細](../detail-design/詳細設計書.md): 「WAF: マネージドルールセット適用」

しかし infrastructure インタビュー（[`infrastructure/interview.md`](../infrastructure/interview.md)）で次の選択がなされた:

- **Q-NET-02（パブリック公開範囲）= B**: API も直接公開（CDN なし）
- **Q-NET-03（WAF / DDoS 対策）= C**: 不要 / 最小限

これは既存設計書の方針と **不整合** な状態となるため、本 ADR で明示的に意思決定を上書き・記録する。

判断材料:

- MVP スコープ・コスト最適化を優先（[tech-stack Q-PRJ-03 = バランス](../tech-stack/decision-record.md)）
- 利用者数が初期は小規模（KPI: MVP DAU 計測のみ）
- アプリ層で OWASP Top 10 対策を実施する方針（[詳細設計書 §7.2](../detail-design/詳細設計書.md)）
- Cloud Run / Cloud Load Balancing には GCP インフラ層での基本的な DDoS 緩和が組み込まれている
- 日記アプリの性質上、悪意ある外部攻撃の標的になりにくい（公開掲示板や決済機能なし）

## Decision

MVP では以下を **不採用** とする:

- **Cloud CDN を使用しない**: Cloud Run を直接 Cloud Load Balancing 経由で公開
- **Cloud Armor を使用しない**: WAF ルールセットを構成しない

代替策（既存対策で代用）:

| 脅威 | 代替策 |
|---|---|
| DDoS | GCP のインフラ層保護（Cloud Load Balancing と Cloud Run の組み込みプロテクション） |
| SQLi / XSS | アプリ層: ORM の prepared statement、入力検証、レスポンスエスケープ |
| 認可エラー悪用 | アプリ層: 全 API で `user_id == session.user_id` 検証（詳細設計書 §7.1） |
| 大量リクエスト | アプリ層レートリミット（詳細設計書 §3.4）+ Cloud Run の同時実行数上限 |
| 静的アセット配信 | Cloud Run の SSR レスポンス + ブラウザキャッシュヘッダ（Cache-Control） |

## Consequences

### Good
- **ランニングコスト削減**: Cloud CDN（リクエスト課金）と Cloud Armor（固定費 + ルール課金）が不要
- **構成のシンプル化**: 管理対象レイヤが減る（CDN キャッシュ無効化やルール調整が不要）
- **障害切り分けが容易**: ネットワーク経路が短く、ボトルネック箇所が少ない

### Bad / Trade-offs
- **オリジン直撃**: 大量アクセス時に Cloud Run に直接負荷がかかる（オートスケールで吸収するが、コストが急増しうる）
- **アプリ層対策に依存**: アプリ層に脆弱性があると即被害（バリデーション抜けが致命的になる）
- **既存設計書（基本・詳細）の方針と不整合**: 本 ADR で上書きするが、設計書側のテキストは古い記述を残す（誤読リスクあり）
- **静的アセット配信が遅い**: CDN ありに比べてレイテンシが大きい（特に海外利用者）

### 既存設計書への影響
- [基本設計書 §8.3](../basic-design/基本設計書.md) の WAF / CDN 記述は **方針レベルの記述として残置**、本 ADR が現在の決定として上位
- [詳細設計書 §8.2](../detail-design/詳細設計書.md) の WAF 記述も同様
- リーダーの誤読防止のため、両ファイルに「現在の方針は ADR 0005 を参照」の注記を追記推奨（フォローアップ）

## Alternatives

| 案 | Pros | Cons | 不採用理由 |
|---|---|---|---|
| **既存設計書通り Cloud Armor + Cloud CDN を採用** | 推奨設定、防御層厚い | コスト増（月額数千円〜数万円）、運用対象増 | MVP コスト要件と不整合 |
| **Cloudflare を前段に挟む** | 安価、グローバルエッジ | マルチベンダーで構成複雑化、Cloud Run と二重管理 | GCP 単独で完結させる方針と矛盾 |
| **Cloud Armor のみ採用（CDN なし）** | DDoS 防御だけ確保 | ルール運用負荷あり | DDoS の現実的脅威度が低いと判断 |
| **Cloud CDN のみ採用（WAF なし）** | キャッシュ効果、レイテンシ改善 | 動的コンテンツ中心の SSR では効果限定的 | 効果対コストの優位性低 |

## 将来の見直しトリガー

以下のいずれかが発生した場合、WAF / CDN の追加を **新規 ADR で再決定** する:

| トリガー | 想定アクション |
|---|---|
| DAU 5,000 超え | パフォーマンス計測のうえ CDN 採用を検討 |
| 急激なトラフィック増を観測 | Cloud Armor のレート制限ルールを先行追加 |
| セキュリティインシデント発生 | WAF を即時導入、攻撃パターンに応じたルール作成 |
| 外部 DDoS の兆候を観測 | Cloud Armor のアダプティブ保護を有効化 |
| 海外利用者比率が一定以上に | Cloud CDN（または Cloudflare）でエッジ配信 |

## フォローアップ

本 ADR を承認後、以下のドキュメントへの注記追加を推奨:

- [基本設計書 §8.3](../basic-design/基本設計書.md) に「※ MVP では ADR 0005 により WAF/CDN を不採用」を追記
- [詳細設計書 §8.2](../detail-design/詳細設計書.md) に同様の注記
