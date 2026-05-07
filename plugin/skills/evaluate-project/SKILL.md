---
name: evaluate-project
description: |
  Evaluate a target project against the agentic quality gate (master.xlsx, 176 entries).
  4-Phase pipeline: Sense (external signals) → Refresh (KB regen) → Evaluate (gate) → Feedback (effectiveness).
  Use when user asks to evaluate, review, audit, or check the quality of a project given its root folder path.
  Examples: "このプロジェクトを評価して", "/path/to/project の品質ゲートを通して", "RFP と計画書をレビューしてレポート出して".
---

# Skill: evaluate-project

評価対象プロジェクト（RFP / 計画書 / コード等）のルートを受け取り、**4 Phase パイプライン** を実行して品質報告書群を発行する。

## 4 Phase パイプライン

```
Phase 0: 外部変化サーチ (NEW)         signal-sensor agent
Phase 1: ナレッジ最新化              excel_to_md.py スクリプト
Phase 2: プロジェクト評価            project-explorer + gate-evaluator + report-writer
Phase 3: フィードバック反映 (NEW)    feedback-collector agent
```

各 Phase の出力は `reports/` に Markdown で残る。

## 起動条件

ユーザーが以下のような表現をした場合：
- 「このプロジェクトを評価して」
- 「○○ をレビューして報告書を出して」
- 「品質ゲート評価を実行」
- 「/aqg:evaluate <path>」

## 必要な情報

### 必須

- **`project_root`**: 評価対象プロジェクトの絶対パス

### オプション

- **`evaluation_scope`**: `rfp_review` / `plan_review` / `code_review` / `all` (default)
- **`target_phases`**: 評価フェーズ（例: `["P0", "P1", "X"]`）
- **`target_severity`**: `["critical", "high"]` がデフォルト
- **`output_path`**: 報告書出力ディレクトリ
- **`skip_sensing`**: `true` でPhase 0 をスキップ（前回sense結果を流用）
- **`skip_feedback`**: `true` でPhase 3 をスキップ（試走時など）
- **`lookback_days`**: Phase 0 の観測期間（デフォルト 30）

## 実行フロー

### Phase 0: 外部変化サーチ

`skip_sensing=true` なら飛ばす。それ以外は：

1. `project_root` から軽量に tech_stack を推定（README/package.json/*.tf 等を Read）
   - 失敗してもよい。空の tech_stack で進める（その場合は信号の関連度は LLM 単独判断）
2. `signal-sensor` agent を Task ツールで起動
   - 入力: project_root, tech_stack, lookback_days, master_xlsx
3. 結果を受け取り、外部変化レポートのパスをメモ
4. ユーザーへの中間報告：
   ```
   📡 Phase 0: 外部変化サーチ完了
   - 観測信号: N件 / 自社関連: M件 / 新規候補: K件
   - レポート: reports/{ts}_signal_report.md
   ```

### Phase 1: ナレッジ最新化

外部変化があり candidates が **既に人間によって promoted されている** 場合（candidates シートで `status=promoted` の行がある）：

- promoted 行を Checklist シートに昇格させる Python スクリプト実行
- その後 `python3 scripts/excel_to_md.py` で Markdown 再生成

通常時（candidates が candidate 状態のまま）：
- ナレッジ最新化はスキップ（人間レビュー待ち）
- ユーザーへの注意：
   ```
   📋 Phase 1: 候補 N件 が人間レビュー待ち（candidates シート）
      → 採用するなら master.xlsx で status を promoted に変更してください
   ```

### Phase 2: プロジェクト評価（既存3エージェント）

旧フローと同じ：

1. `project-explorer` agent を Task で起動 → exploration_report
2. `gate-evaluator` agent を Task で起動 → evaluation_result
3. `report-writer` agent を Task で起動 → report.md

複数の評価対象（RFP/計画書/コード）が同一 root にある場合は、3並列で起動（前回実装通り）。

評価結果のJSONは `/tmp/aqg_eval_result.json` に保存（Phase 3 で使う）。

### Phase 3: フィードバック反映

`skip_feedback=true` なら飛ばす。それ以外は：

1. `feedback-collector` agent を Task で起動
   - 入力: evaluation_result の JSON path, master_xlsx, project_name
2. master.xlsx の effectiveness シートが更新される
3. ユーザーへの中間報告：
   ```
   🔁 Phase 3: フィードバック反映完了
   - effectiveness シート: 更新 N件 / 新規 M件
   - 要レビュー (FP率>30%): K件
   ```

## ユーザーへの最終報告

4 Phase 完了後、以下をテキスト出力：

```
✅ 品質ゲート評価完了（4 Phase）

【Phase 0】外部変化サーチ
- 観測信号: N件 / 自社関連: M件 / 新規候補: K件 / 既存更新候補: J件
- レポート: reports/{ts}_signal_report.md

【Phase 1】ナレッジ最新化
- 採用済 candidates: N件 → Checklist へ昇格
- 候補待ち: M件（人間レビュー対象）
- Markdown再生成: 176 件 → ?? 件

【Phase 2】プロジェクト評価
- 報告書: reports/{ts}_{project}_review.md
- 総合判定: <Pass | Conditional | Fail>
- Critical Fail: N件 / Human レビュー必須: M件

【Phase 3】フィードバック反映
- effectiveness 更新: N件
- 要レビュー (FP率>30%): K件
- サマリ: reports/{ts}_feedback.md

主要リスク（Top 3）:
1. ...
2. ...
3. ...

次のアクション:
→ 評価報告書を開く（Phase 2）
→ 外部変化レポートを確認、候補の採用判断（Phase 0）
→ effectiveness で要レビュー項目を Curator agent で再検証（Phase 3）
```

## エラー処理

- Phase 0 で WebFetch 失敗 → 信号取得失敗を記録、Phase 1 以降は通常実行
- Phase 1 で master.xlsx 読込失敗 → エラー報告、ユーザーに master.xlsx 確認を求める
- Phase 2 で project_root が空 → 通常通り評価できない旨を報告
- Phase 3 で集計失敗 → 評価結果は保持、後で再実行可能と案内

## 設計原則の確認

- **AI自律探索**: ルートパスだけ指定で進む
- **柔軟判断**: ナレッジ176件への適用判断はLLM文脈推論
- **引用必須**: 全判定に file:line またはdoc:section の引用付き
- **Human-in-Loop**: candidate 採用判断と Conditional/Critical Fail のレビュー
- **生きた仕様**: 評価のたびに master.xlsx に学習が蓄積、ナレッジが進化
