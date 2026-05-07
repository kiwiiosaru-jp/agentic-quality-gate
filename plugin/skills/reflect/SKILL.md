---
name: reflect
description: |
  Run the Reflective Curator: analyze accumulated incidents, false-positive metrics, and past evaluation patterns to derive new/updated knowledge candidates. Layer 6 internal learning loop.
  Examples: "内省サイクルを実行", "/aqg:reflect", "蓄積した経験からナレッジ候補を作って", "FP率の高い項目を見直して".
---

# Skill: reflect

自社経験（incidents シート + effectiveness シート + 過去評価レポート）を内省的に分析し、`candidates` シートにナレッジ候補を派生させる。

## 起動条件

- 「内省サイクルを実行」
- 「蓄積した経験からナレッジ候補を作って」
- 「FP率高い項目を見直して」
- 「/aqg:reflect」

## 入力

### 必須

なし。デフォルトで動く（mode=all）。

### オプション

- **`mode`**: 
  - `all` (default) — 4モード全部
  - `incidents-only` — Mode 1 のみ（incidentsの抽象化）
  - `effectiveness-only` — Mode 2 のみ（FP率高エントリの曖昧さ検出）
  - `conditional-pattern` — Mode 3 のみ（Conditional観点抽出）
  - `cross-trend` — Mode 4 のみ（横断トレンド）
- **`lookback_days`**: 何日前までの incidents を見るか（デフォルト 90）
- **`fp_threshold`**: FP率の閾値（デフォルト 30%）
- **`min_pattern_count`**: 抽象化の最小件数（デフォルト 2）

## 実行手順

`reflective-curator` Subagent を Task で起動：

```
入力:
  master_xlsx: $CLAUDE_PLUGIN_ROOT/knowledge/master.xlsx
  mode: <指定 or "all">
  lookback_days: 90
  fp_threshold: 30
  min_pattern_count: 2
```

## 出力

```
✅ Reflective Curator 実行完了

📊 入力データ
- Incidents (raw, 過去90日): N件
- FP率高エントリ (>30%): M件
- 過去評価レポート Conditional観点: K件

🎯 派生 candidates: 合計 N件
- Mode 1 (Incidents 抽象化): N件
- Mode 2 (FP曖昧さ): M件
- Mode 3 (Conditional観点): K件
- Mode 4 (横断トレンド): J件

レポート: $CLAUDE_PLUGIN_ROOT/reports/{ts}_reflective.md
master.xlsx 更新:
- candidates シート: N件追加
- incidents シート: M件 raw → processed
- reflections シート: 1件追加（実行履歴）

次のアクション:
→ master.xlsx の candidates シートで人間レビュー
→ 採用するなら status を 'promoted' に変更
→ 次回 /aqg:evaluate で Phase 1 で Checklist へ昇格
```

## ユースケース

| シーン | 推奨モード | 推奨頻度 |
|---|---|---|
| 月次レビュー会議の前 | `all` | 月1回 |
| FP率の高さに気づいた | `effectiveness-only` | 随時 |
| 大きなインシデント直後 | `incidents-only` | インシデントごと |
| 評価結果の振り返り | `conditional-pattern` | 評価3-5回ごと |
| 業界動向と自社経験を相関 | `cross-trend` | 四半期ごと |

## 設計上のポイント

- **過剰な一般化を避ける**: 1件のincidentで抽象化しない（min_pattern_count=2）
- **採用判断は人間**: 候補を出すだけ、Checklist への昇格は手動
- **証跡保持**: 各候補の rationale に「どのincident/どのreport」由来かを必ず記載
- **incidents 状態管理**: 処理済みは raw → processed、再処理を防止

## 関連スキル

- `/aqg:incident` — 経験を記録（reflectの入力データを蓄積）
- `/aqg:evaluate` — 評価実行（Phase 3 で effectiveness 自動更新）
- `/aqg:sense` — 外部信号取込（Reactive、reflectとは独立）
