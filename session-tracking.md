# 学習時間の記録

Copyright (c) 2026 Tsuyoshi Hemmi — All Rights Reserved

冒険者が学習にかけた時間を記録し、ステータス画面で振り返れるようにする。**記録は完全にローカル**で、API には送信しない。

SKILL.md 本体はこの仕組みの存在と書き込みタイミングのトリガーのみ把握する。詳細（イベントスキーマ・集計ルール・異常終了対応）は本ファイルを参照する。

---

## 使用する 2 つのファイル

### `~/.config/java-quest/sessions.log`

JSONL（1 行 1 JSON）形式の **追記専用** ログ。**state-change イベントのみ** を記録する（通信 1 回ごとの書き込みはしない）。

書き込むイベント種別:

| event | 補足フィールド | 書き込みタイミング |
|-------|---------------|------------------|
| `session_start` | `session_id` (UUID v4) | `/java-quest` 起動が STEP 3 まで成功した直後 |
| `dungeon_enter` | `dungeon_id` | ダンジョン選択確定時 |
| `dungeon_exit` | `dungeon_id`, `reason` | ホームに戻る・別ダンジョンへ切替・session_end 時 |
| `exercise_result` | `dungeon_id`, `result` (`pass`/`fail`) | 通常演習の採点結果確定時 |
| `boss_result` | `dungeon_id`, `result` (`defeat`/`fail`) | ボス採点結果確定時 |
| `session_end` | — | 「4. 冒険を中断する」選択時 |

すべての行に共通して `ts`（ISO8601、UTC）と `uuid`（冒険者UUID）を含める。

`dungeon_exit.reason` の値: `home` | `switched`（別ダンジョンへ移動） | `session_end`

### 例

```jsonl
{"ts":"2026-04-14T10:00:00Z","uuid":"c536190f-...","event":"session_start","session_id":"a1b2..."}
{"ts":"2026-04-14T10:02:30Z","uuid":"c536190f-...","event":"dungeon_enter","session_id":"a1b2...","dungeon_id":"C1-01"}
{"ts":"2026-04-14T10:25:10Z","uuid":"c536190f-...","event":"exercise_result","session_id":"a1b2...","dungeon_id":"C1-01","result":"pass"}
{"ts":"2026-04-14T10:40:00Z","uuid":"c536190f-...","event":"dungeon_exit","session_id":"a1b2...","dungeon_id":"C1-01","reason":"home"}
{"ts":"2026-04-14T10:40:05Z","uuid":"c536190f-...","event":"session_end","session_id":"a1b2..."}
```

### `~/.config/java-quest/last_activity.json`

直近活動のスナップショット。**state-change のたびに上書き更新**する。

```json
{
  "session_id": "a1b2...",
  "started_at": "2026-04-14T10:00:00Z",
  "last_activity_at": "2026-04-14T10:25:10Z",
  "current_context": "dungeon",
  "current_dungeon_id": "C1-01"
}
```

`current_context` の値: `home` | `dungeon` | `boss` | `ended`

---

## 書き込みルール（実装指針）

1. イベントが発生したら **まず `sessions.log` に追記**、次に **`last_activity.json` を上書き**
2. 書き込み失敗は致命的ではない — エラーをログに出すのみで Skill は停止しない（学習時間は付随機能のため）
3. `sessions.log` のサイズが肥大化しても（想定年間数 MB 程度）ローテーションは行わない。手動削除は可

---

## 集計ルール（ステータス画面用）

`sessions.log` をパースして以下を計算する:

- **1 セッションの長さ** = `session_end.ts - session_start.ts`
  - `session_end` が無いセッションは `last_activity.json.last_activity_at` を終了時刻として使う（後述の異常終了ルール参照）
- **今日** = `session_start.ts` がローカル日付の本日 0:00 以降のセッション長の合計
- **今週** = 直近 7 日間のセッション長の合計 ＋ そのセッション数
- **累計** = 全セッションの長さの合計
- **ダンジョン別時間** = 同一 `session_id` 内の `dungeon_enter` → `dungeon_exit` ペアの差分を `dungeon_id` ごとに集計。上位 3 件を表示（同点は dungeon_id 昇順）

表示フォーマットは `{H}時間{M}分`（分未満切り捨て、0 時間なら `{M}分` のみ）。データがまだ無い場合は各欄 `0分` を表示。

---

## 異常終了の扱い（暫定仕様）

Claude Code が突然閉じる等で `session_end` が書けなかった場合の集計ルール。**正式決定までは以下の暫定扱い**。

- 集計時、`session_start` に対応する `session_end` が無いセッションは `last_activity.json.last_activity_at` を終了時刻として使う
- `last_activity_at` も古い（例: 24 時間以上前）場合はそのセッションを集計から除外し、次回起動時にリーナが「前回のセッションは記録が不完全です」と軽く触れる

この暫定仕様は java-quest Issue 側で追跡予定。変更の可能性あり。
