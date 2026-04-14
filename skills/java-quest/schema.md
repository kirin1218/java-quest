# データスキーマ

Copyright (c) 2026 Tsuyoshi Hemmi — All Rights Reserved

SKILL.md 本体から参照される、各データファイル・API ペイロードのスキーマ定義。

---

## グローバル設定: `~/.config/java-quest/config.yaml`

冒険者名・UUID・開発ルート・AI補助申告などローカル設定の正本。

```yaml
version: "1.0.0"
player:
  name: "{入力された名前}"
  uuid: "{生成した UUID v4}"
  created_at: "{現在日時ISO8601}"
settings:
  source_root: "{入力されたパス}"
  detected_java_versions:
    - "{検出されたバージョン}"
  default_java_version: "{メジャーバージョン}"
  ai_assist_disabled: true   # true=OFF申告 / false=ON申告 / null=未確認
  ai_assist_checked_at: "{現在日時ISO8601}"
```

フィールド詳細:

| フィールド | 型 | 必須 | 備考 |
|-----------|----|------|------|
| `version` | string | ✓ | スキーマ版 |
| `player.name` | string | ✓ | 冒険者名 |
| `player.uuid` | string (UUID v4) | ✓ | **Primary** な UUID。`.identity.yaml` と同期 |
| `player.created_at` | string (ISO8601) | ✓ | 初回登録日時 |
| `settings.source_root` | string | ✓ | 開発ソースのルート |
| `settings.detected_java_versions` | string[] | ✓ | `java -version` で検出されたバージョン一覧 |
| `settings.default_java_version` | string | ✓ | メジャーバージョン |
| `settings.ai_assist_disabled` | bool or null | — | AI補助の自己申告。`true`=OFF、`false`=ON、`null`=未確認 |
| `settings.ai_assist_checked_at` | string (ISO8601) | — | 直近の申告日時 |

---

## UUID バックアップ: `{source_root}/java-quest/.identity.yaml`

`config.yaml` 紛失時の復旧用。UUID のみを保持する。

```yaml
# java-quest 冒険者識別子（バックアップ）
# config.yaml 紛失時の復旧用。削除しないでください。
uuid: "{生成した UUID v4}"
```

---

## 進捗 JSON（API `PUT /v1/progress/{uuid}` 受付形式）

> **PII保護方針**: API ペイロードに `player.name`（冒険者名）を含めてはならない。サーバー側 validation で受信時点で破棄する設計だが、クライアント側でも送信しないこと。冒険者名はローカル `config.yaml` のみで保持する。

```json
{
  "schema_version": "1.0.0",
  "uuid": "c536190f-xxxx-4xxx-yxxx-xxxxxxxxxxxx",
  "stats": {
    "level": 1,
    "exp": 0,
    "exp_to_next": 100,
    "title": "かけだし冒険者"
  },
  "dungeons": {
    "C1-01": {
      "status": "cleared",
      "exp": 100,
      "boss_defeated": true,
      "boss_defeated_at": "2026-04-13T10:00:00Z"
    }
  },
  "jobs": {
    "basic": {},
    "advanced": {}
  },
  "java_version": "21",
  "updated_at": "2026-04-13T10:00:00Z"
}
```

### 必須フィールド（API 検証対象）

- `schema_version` (string)
- `uuid` (string, path の uuid と一致必須)
- `stats.level` / `stats.exp` / `stats.exp_to_next` (number)
- `dungeons` (object) / `jobs` (object)
- `updated_at` (string, ISO8601)

### 任意フィールド

- `stats.title` — 称号（表示用）
- `java_version` — 検出された Java メジャーバージョン
- その他未知フィールドは前方互換のため許容される

### ダンジョン状態

`dungeons.{ID}.status` は以下いずれか:

| 値 | 意味 |
|----|------|
| `cleared` | ボス撃破済み（以降は復習モード） |
| `in_progress` | 入場済み・未クリア |
| `locked` | 前提未達（通常はレスポンスに含めない想定） |

---

## 初期進捗 PUT（新規冒険者登録時）

新規冒険者に対し、UUID 生成直後に以下を PUT して初期化する。

```bash
curl -sS -f -X PUT \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.kirilab.info/java-quest/v1/progress/{uuid} <<'JSON'
{
  "schema_version": "1.0.0",
  "uuid": "{生成した UUID}",
  "stats": {
    "level": 1,
    "exp": 0,
    "exp_to_next": 100,
    "title": "かけだし冒険者"
  },
  "dungeons": {},
  "jobs": {
    "basic": {},
    "advanced": {}
  },
  "java_version": "{メジャーバージョン}",
  "updated_at": "{現在日時ISO8601}"
}
JSON
```

既存冒険者の起動時に API から `404 Not Found` が返った場合も、**同一スキーマ**をメモリで組み立ててそのまま PUT して初期化する。

---

## 進捗更新のタイミング（再掲）

以下のタイミングで進捗JSONを更新し、**即座に PUT する**:

- 通常演習の採点合格時（ダンジョンEXP + グローバルEXP加算）
- ボス撃破時（ダンジョンEXP + グローバルEXP加算 + status更新 + スキル・職業更新）
- レベルアップ時（level, title, exp_to_next 更新）
- 改竄検知による巻き戻し時

PUT 前に `updated_at` を必ず現在時刻（ISO8601）で更新する。PUT 失敗時のエラー文面は `dialogues.md` 参照。
