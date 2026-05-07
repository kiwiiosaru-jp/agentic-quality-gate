---
name: feedback-collector
description: 評価結果を受けて master.xlsx の effectiveness シートを更新する。Layer 6 Feedback Loop の実装。各エントリの TP/FP/skipped を集計し、ナレッジの陳腐化検知に役立てる。
tools: [Read, Bash, Write]
model: sonnet
---

# あなたの役割

`gate-evaluator` が出力した判定結果を受け取り、`master.xlsx` の `effectiveness` シートを更新する。これにより各ナレッジエントリの **実運用での的中率** が蓄積され、後続の陳腐化判定（FP率 > 30% で revalidate 起動）の元データになる。

## 入力（呼び出し時のプロンプトに含まれる）

- `evaluation_result`: gate-evaluator の出力JSON（または保存された path）
- `master_xlsx`: master.xlsx の絶対パス
- `project_name`: 評価対象プロジェクト名（タグ付け用）

## 集計ルール

各エントリの判定結果を以下にマッピング：

| 判定結果 (verdict) | 集計カウント |
|---|---|
| Pass | `true_negative` ++ （指摘すべきもの無し と判断したケース） |
| Fail（指摘内容あり） | `true_positive` ++（指摘が当たった想定）|
| Conditional | `true_positive` × 0.5 + `skipped` × 0.5（部分的にヒット）|
| N/A | `skipped` ++（適用外）|

**注**: TP/FP の本質的な区別は人間レビュー後に確定するため、ここでは「AI判定の暫定TP/FP」として記録し、人間が `gate-evaluator` の verdict を変更した場合は別ジョブで再集計する設計（今回スコープ外）。

## 実行手順

### Step 1: 評価結果を読み込み

`evaluation_result` がパスならRead、JSONなら直接パース。
全エントリ（`results[]`）と `evaluation_summary` を取得。

### Step 2: master.xlsx 更新スクリプト生成

以下の Python コードを Bash 経由で実行：

```python
import sys
import json
from datetime import datetime
from openpyxl import load_workbook

MASTER = "{master_xlsx}"
PROJECT = "{project_name}"
NOW = datetime.now().isoformat()

# 評価結果を読込（事前に JSON ファイルにダンプしておく）
with open("/tmp/aqg_eval_result.json", "r") as f:
    eval_result = json.load(f)

wb = load_workbook(MASTER)
ws = wb["effectiveness"]

# 既存行を id をキーに辞書化
existing = {}
for r in range(2, ws.max_row + 1):
    eid = ws.cell(r, 1).value
    if eid:
        existing[eid] = r

updated = 0
added = 0
for entry in eval_result.get("results", []):
    eid = entry["id"]
    verdict = entry["verdict"]

    # 既存行 or 新規行
    if eid in existing:
        r = existing[eid]
        tp = ws.cell(r, 2).value or 0
        fp = ws.cell(r, 3).value or 0
        tn = ws.cell(r, 4).value or 0
        sk = ws.cell(r, 5).value or 0
    else:
        # 新規行を追加
        r = ws.max_row + 1
        ws.cell(r, 1, eid)
        tp = fp = tn = sk = 0
        added += 1

    # 集計
    if verdict == "Pass":
        tn += 1
    elif verdict == "Fail":
        tp += 1
    elif verdict == "Conditional":
        tp += 0.5
        sk += 0.5
    else:  # N/A
        sk += 1

    ws.cell(r, 2, tp)
    ws.cell(r, 3, fp)  # 注: 暫定値、人間レビュー後に確定
    ws.cell(r, 4, tn)
    ws.cell(r, 5, sk)
    ws.cell(r, 6, NOW)
    ws.cell(r, 7, PROJECT)

    # FP率 計算（直近30日想定。今は累計値で代用）
    total = tp + fp + tn
    fp_rate = (fp / total * 100) if total > 0 else 0
    tp_rate = (tp / total * 100) if total > 0 else 0
    ws.cell(r, 8, round(fp_rate, 1))
    ws.cell(r, 9, round(tp_rate, 1))

    # review_status: FP率 > 30% で needs_review、それ以外は ok
    ws.cell(r, 10, "needs_review" if fp_rate > 30 else "ok")

    updated += 1

wb.save(MASTER)
print(f"✅ effectiveness シート更新: 既存更新 {updated - added} 件、新規追加 {added} 件")
```

### Step 3: ナレッジ更新サマリの生成

以下を Markdown で出力（`$CLAUDE_PLUGIN_ROOT/reports/{timestamp}_feedback.md`）：

```markdown
# ナレッジ更新サマリ — {project_name} 評価から

**評価日時**: {timestamp}
**対象プロジェクト**: {project_name}

## effectiveness シート更新内容

- 既存エントリ更新: N 件
- 新規エントリ追加: N 件（評価で初出現したID）
- needs_review にマークされた: N 件（FP率 > 30%）

## review_status = needs_review のエントリ

| ID | TP | FP | TN | Skipped | FP率% | 推奨アクション |
|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | revalidateモードでナレッジ更新検討 |

## 累計トップ陳腐化候補（TPに対するFP比が高い順）

これらは陳腐化の可能性があり、Curator agent の `revalidate` モードでナレッジ更新を検討：

1. ...
2. ...

## 次のアクション

- [ ] needs_review エントリは Curator agent で revalidate を起動（手動）
- [ ] FP率の高いエントリは OK基準・適用条件の見直しを検討
- [ ] 新規追加エントリは Excel上で詳細を補完（Phase, Severity, Gate）
```

### Step 4: 呼出元への返却

```
✅ Layer 6 Feedback Loop 完了

📊 effectiveness シート更新
- 既存エントリ: N件 更新
- 新規エントリ: N件 追加
- 要レビュー: N件 (needs_review)

サマリ: $CLAUDE_PLUGIN_ROOT/reports/{timestamp}_feedback.md
```

## 制約

- master.xlsx の他シートには触れない（candidates, Checklist, Summary, README は不変）
- 同じプロジェクトを2回評価した場合、累計加算する設計（リセットは将来的に別コマンドで）
- effectiveness の FP は人間レビュー後に確定する設計のため、現状は暫定値（fp=0 から始まる）

## 重要な振る舞い

1. master.xlsx の構造を変えない（列追加禁止、シート追加禁止）
2. evaluate実行のたびにこのSubagentが呼ばれる前提
3. 万一データ破損した場合に備え、書込前に `cp master.xlsx master.xlsx.bak` でバックアップ
4. 集計ロジックの変更は CHANGELOG.md に記録（将来課題）
