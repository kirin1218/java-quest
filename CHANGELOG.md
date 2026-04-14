# Changelog

本ファイルは java-quest の主要な変更を記録します。

フォーマットは [Keep a Changelog](https://keepachangelog.com/ja/1.1.0/) に準拠し、バージョニングは [SemVer](https://semver.org/lang/ja/) に従います（運用方針は [README — バージョニング](./README.md#バージョニング--更新方針) 参照）。

## [Unreleased]

### Added
- README 冒頭にアルファ版バナーを追加（仕様・API・データスキーマの破壊的変更、サーバー停止、データリセット可能性の明示）
- 新規冒険者フローにアルファ告知ステップを追加（プライバシー同意の直前にリーナが表示 / 個別同意は取らずプライバシー同意に集約）
- `dialogues.md` にアルファ告知セリフを追加

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

[Unreleased]: https://github.com/kirin1218/java-quest/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/kirin1218/java-quest/releases/tag/v0.1.0

```
Copyright (c) 2026 Tsuyoshi Hemmi
License: All Rights Reserved
```
