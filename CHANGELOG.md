# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### 0.1.0 — 初版公開（実装込み）
- Apache-2.0 ライセンスでリポジトリ公開
- `plugin/` に Claude Code Plugin 一式投入（匿名化済み）
  - 8 Subagent（gate-evaluator, signal-sensor, reflective-curator, feedback-collector, incident-recorder, project-explorer, report-writer, claude-security-adapter スケルトン）
  - 6 Skill（checklist, evaluate-project, reflect, sense, incident, claude-security スケルトン）
  - 6 Slash Command（/aqg:checklist, /aqg:evaluate, /aqg:reflect, /aqg:sense, /aqg:incident, /aqg:claude-security）
  - 176 件のナレッジ（master.xlsx + Markdown 自動派生 177 件）
  - 4 Script（excel_to_md, add_meta_sheets, add_incidents_sheet, add_claude_security_sheet）
- `skill-pm-blueprint/` に pm-blueprint Skill 投入（9 レイヤ、テンプレート 19 種、架空企業適用例）
- `docs/architecture.md` に全体アーキ概要

### 後続リリース予定
- **Claude Security 本実装**（現状はスケルトンのみ）：Subagent / Hook の本体、判定ロジック、評価セット連携
- `docs/` 拡充：QuickStart、phase-gates 詳細仕様、Claude Code プリミティブとのマッピング

### 匿名化処理
- 客先名（佐川急便等）、客先固有スタック名（Snowflake/Databricks/Cortex 等）、規模数値、個人パスを全削除
- 内部評価結果（reports/）と内部変更履歴（IMPROVEMENT-*.md）は除外
- master.xlsx の「由来」列は内部メモのため削除（25列 → 24列）

---

## [0.0.1-skeleton] - 2026-05-07

### Added
- 初期構造（README, LICENSE, CHANGELOG, 4 サブディレクトリ）
