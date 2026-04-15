# Changelog

本ファイルは java-quest の主要な変更を記録します。

フォーマットは [Keep a Changelog](https://keepachangelog.com/ja/1.1.0/) に準拠し、バージョニングは [SemVer](https://semver.org/lang/ja/) に従います（運用方針は [README — バージョニング](./README.md#バージョニング--更新方針) 参照）。

## [Unreleased]

## [0.1.2] — 2026-04-15 — Fix scaffolding regression

### Fixed
- 通常演習・ボス課題で課題フォルダと骨格 `.java` が自動生成されない不具合を修正（SKILL.md 圧縮（PR #12）以降、文章短縮で「誰がファイルを作るか」が曖昧になり冒険者に丸投げされていた）— 手順を「Bash で mkdir → Write で README → Write で骨格 `.java`」に明示化し、エリア別の骨格内容を表で固定

## [0.1.1] — 2026-04-15 — Alpha consent dialogue & doc polish

`/plugin update` のキャッシュ更新を確実に走らせるため、本リリース以降アルファ期は修正のたびに patch をインクリメントする。

### Fixed
- プライバシー同意画面のセリフ（`dialogues.md`）を PRIVACY.md と整合させた — 冒険者名をサーバー保存項目から除外し、IP アドレスは運用目的のみで UUID と紐付けない旨を明示
- マーケットプレイス `source` 形式と `plugin.json` スキーマを Claude Code 公式形式に修正（HTTPS url 形式 / repository 文字列形式）

### Added
- README 冒頭にアルファ版バナーを追加（仕様・API・データスキーマの破壊的変更、サーバー停止、データリセット可能性の明示）
- 新規冒険者フローにアルファ告知ステップを追加（プライバシー同意の直前にリーナが表示 / 個別同意は取らずプライバシー同意に集約）
- `dialogues.md` にアルファ告知セリフを追加
- README に「ロードマップ」節を追加（アルファ / ベータ / 安定版の3段階スコープ）
- README に「既知の制限」節を追加（カリキュラム範囲、認証なし、レート制限、改竄検知の限界、AI 補助 OFF 自己申告、Java 環境前提、ネットワーク必須）
- README に「フィードバック・問い合わせ」節を追加（不具合報告・削除請求・GDPR 対応・ライセンス問い合わせ窓口を集約、PRIVACY.md / LICENSE への相互リンク付き）
- README ロードマップ節に「将来的な有償化の可能性」の注記を追加（控えめに明示、方針変更時は CHANGELOG / README で事前告知する旨を記載）
- README「ライセンス / クレジット」節冒頭に免責一文を追加（既存商業 RPG 作品との無関係および独自設計の明示）
- README ロードマップ節の有償化注記に「事業として提供されるものではない」旨を追記（消費者契約法 8 条の事業性認定予防）
- SKILL.md STEP 4 に改竄検知が best effort である旨を明記（README 既知の制限との整合）
- SKILL.md「進捗更新ルール」節に PII 保護方針を再掲（PUT ペイロードに `player.name` を含めない旨の二重防御）

### Changed
- LICENSE 第 2 項(d) を著作物ベースの禁止表現に修正（名称・登場人物名単体の使用禁止から、本ソフトウェア付属の著作物の本ソフトウェア外使用禁止へ）

## [0.1.0] — 2026-04-15 — Alpha initial

初の公開アルファリリース。MVP として「はじまりの平原」「分岐の森」（C1-01〜C1-12 / 12 ダンジョン）と剣士・武闘家・狂戦士の職業システムを提供します。

### Added
- Claude Code スキル本体（`/java-quest`）— AI チューター「リーナ」が進行する RPG 型 Java 学習体験
- 12 ダンジョン（C1-01〜C1-12）のカリキュラム — Java 基本構文・制御構文を講義 → 演習 → ボス戦の3段階で習得
- 職業システム — 基本職（剣士・武闘家）と上級職（狂戦士）
- レベル・称号システム — Lv.1（かけだし冒険者）〜 Lv.5（ループの旅人）
- 配信 API（`api.kirilab.info/java-quest/v1/`）— カリキュラム配信と進捗保存
- ローカル設定 `~/.config/java-quest/config.yaml` と UUID バックアップ `.identity.yaml`
- 学習時間トラッキング（ローカル `sessions.log` ベース）
- 答え直接要求の自動リジェクト機能（`rejection-patterns.md`）
- 改竄検知（提出コードと進捗 JSON の整合チェック）
- Claude Code Marketplace 対応（`/plugin install` で導入可能）
- ロゴ（生成 AI 製作 — ドット RPG 風タイトル）
- 公開ドキュメント一式 — README / LICENSE（カスタムライセンス）/ PRIVACY.md / CHANGELOG.md
- 新規冒険者フローへのプライバシーポリシー同意ステップ

### Security
- API ペイロードから `player.name`（冒険者名）を受信時点で破棄するサーバー側ガード — 冒険者名はローカルのみ保持され、サーバーには保存されない
- API アクセスログ（IP アドレス）と学習進捗データ（UUID）は紐付け解析しない運用方針

### Documentation
- バージョニング方針（0.x = アルファ／ベータ、1.0.0 = 安定版）を README に明記
- DAG 整合性検証レポート（全 65 ダンジョンの前提条件）
- レベルテーブル・英語名規約

---

[Unreleased]: https://github.com/kirin1218/java-quest/compare/v0.1.2...HEAD
[0.1.2]: https://github.com/kirin1218/java-quest/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/kirin1218/java-quest/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/kirin1218/java-quest/releases/tag/v0.1.0

```
Copyright (c) 2026 Tsuyoshi Hemmi
License: All Rights Reserved
```
