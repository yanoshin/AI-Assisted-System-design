# テーブル定義書

PostgreSQL 16 を想定。論理 ER 図は [`basic-design/er/`](../../basic-design/er/) を参照。

## 共通方針

- PK は `bigint generated always as identity`
- 文字列は `text`（長さ制約はアプリ層）
- 日時は `timestamptz`（タイムゾーン付き、UTC で保存）
- 日付は `date`
- 全テーブル外部参照は `ON DELETE CASCADE`（退会時に一括削除）

---

## `users` — 利用者

| カラム | 型 | NULL | デフォルト | 制約 / 備考 |
|---|---|---|---|---|
| `id` | bigint | NOT NULL | identity | PK |
| `display_name` | text | NOT NULL | — | 1〜50 文字（アプリ層） |
| `email` | text | NOT NULL | — | IdP 由来。形式は IdP に委ねる |
| `created_at` | timestamptz | NOT NULL | `now()` | — |

### 索引
| 名前 | 種別 | カラム | 用途 |
|---|---|---|---|
| `users_pkey` | PRIMARY KEY | `id` | — |

### DDL
```sql
CREATE TABLE users (
  id           bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  display_name text        NOT NULL,
  email        text        NOT NULL,
  created_at   timestamptz NOT NULL DEFAULT now()
);
```

---

## `sso_identities` — SSO 連携

| カラム | 型 | NULL | デフォルト | 制約 / 備考 |
|---|---|---|---|---|
| `id` | bigint | NOT NULL | identity | PK |
| `user_id` | bigint | NOT NULL | — | FK → `users(id)` ON DELETE CASCADE |
| `provider` | text | NOT NULL | — | `'google'`。CHECK で値を限定 |
| `provider_subject` | text | NOT NULL | — | IdP での一意 ID (`sub` クレーム) |
| `linked_at` | timestamptz | NOT NULL | `now()` | — |

### 索引・制約
| 名前 | 種別 | カラム | 用途 |
|---|---|---|---|
| `sso_identities_pkey` | PRIMARY KEY | `id` | — |
| `sso_identities_provider_subject_key` | UNIQUE | `(provider, provider_subject)` | IdP の同一サブジェクトを1利用者に束縛 |
| `sso_identities_user_id_idx` | INDEX | `user_id` | 利用者からの逆引き |

### DDL
```sql
CREATE TABLE sso_identities (
  id               bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id          bigint      NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  provider         text        NOT NULL CHECK (provider IN ('google')),
  provider_subject text        NOT NULL,
  linked_at        timestamptz NOT NULL DEFAULT now(),
  UNIQUE (provider, provider_subject)
);
CREATE INDEX sso_identities_user_id_idx ON sso_identities (user_id);
```

---

## `sessions` — セッション

| カラム | 型 | NULL | デフォルト | 制約 / 備考 |
|---|---|---|---|---|
| `id` | bigint | NOT NULL | identity | PK |
| `user_id` | bigint | NOT NULL | — | FK → `users(id)` ON DELETE CASCADE |
| `token_hash` | text | NOT NULL | — | Cookie `sid` の SHA-256 ハッシュ。原文不可 |
| `issued_at` | timestamptz | NOT NULL | `now()` | — |
| `expires_at` | timestamptz | NOT NULL | — | issued_at + 30日 |
| `revoked_at` | timestamptz | NULL | NULL | ログアウト or 退会で記録 |

### 索引・制約
| 名前 | 種別 | カラム | 用途 |
|---|---|---|---|
| `sessions_pkey` | PRIMARY KEY | `id` | — |
| `sessions_token_hash_key` | UNIQUE | `token_hash` | セッション識別 |
| `sessions_user_id_idx` | INDEX | `user_id` | 利用者からの逆引き |
| `sessions_expires_at_idx` | INDEX | `expires_at` | BAT-01 掃除 |

### DDL
```sql
CREATE TABLE sessions (
  id         bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id    bigint      NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token_hash text        NOT NULL UNIQUE,
  issued_at  timestamptz NOT NULL DEFAULT now(),
  expires_at timestamptz NOT NULL,
  revoked_at timestamptz
);
CREATE INDEX sessions_user_id_idx    ON sessions (user_id);
CREATE INDEX sessions_expires_at_idx ON sessions (expires_at);
```

---

## `diary_entries` — 日記エントリ

| カラム | 型 | NULL | デフォルト | 制約 / 備考 |
|---|---|---|---|---|
| `id` | bigint | NOT NULL | identity | PK |
| `user_id` | bigint | NOT NULL | — | FK → `users(id)` ON DELETE CASCADE |
| `entry_date` | date | NOT NULL | — | 日記の日付。1日1エントリ制約 |
| `title` | text | NOT NULL | — | 1〜100 文字（アプリ層） |
| `body` | text | NOT NULL | — | 1〜10,000 文字（アプリ層） |
| `created_at` | timestamptz | NOT NULL | `now()` | — |
| `updated_at` | timestamptz | NOT NULL | `now()` | UPDATE 時に再設定 |

### 索引・制約
| 名前 | 種別 | カラム | 用途 |
|---|---|---|---|
| `diary_entries_pkey` | PRIMARY KEY | `id` | — |
| `diary_entries_user_date_key` | UNIQUE | `(user_id, entry_date)` | **BR-01 1日1エントリ制約** |
| `diary_entries_user_date_desc_idx` | INDEX | `(user_id, entry_date DESC)` | 一覧取得（降順） |

### DDL
```sql
CREATE TABLE diary_entries (
  id         bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id    bigint      NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  entry_date date        NOT NULL,
  title      text        NOT NULL,
  body       text        NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (user_id, entry_date)
);
CREATE INDEX diary_entries_user_date_desc_idx
  ON diary_entries (user_id, entry_date DESC);
```

---

## 補足

### `updated_at` の自動更新（トリガー例）
```sql
CREATE OR REPLACE FUNCTION set_updated_at() RETURNS trigger AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER diary_entries_set_updated_at
  BEFORE UPDATE ON diary_entries
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

### 将来拡張への影響
- `EntryRevision` 追加時: `diary_entries` 本体は変更不要、`UPDATE` 時に履歴を挿入するトリガー or アプリ層で対応
- 複数 IdP 対応時: `sso_identities.provider` の `CHECK` 制約を緩める / マスタテーブル化
