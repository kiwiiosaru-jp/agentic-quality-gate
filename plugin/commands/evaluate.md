---
description: Run agentic quality gate evaluation on a project. Usage- /aqg:evaluate <project-root> [--scope=...] [--severity=...]
allowed-tools: Bash, Glob, Grep, Read, Write, Task
---

`/aqg:evaluate` コマンド：プロジェクトの品質ゲート評価を実行。

引数: `$ARGUMENTS`

## 使い方

```
/aqg:evaluate <project-root>
/aqg:evaluate <project-root> --scope=plan_review
/aqg:evaluate <project-root> --scope=code_review --severity=critical,high
/aqg:evaluate <project-root> --recheck   # 直前の評価を再実行
```

## 動作

1. 引数から `project_root` と オプションを抽出
2. `evaluate-project` skill を呼び出して評価パイプラインを実行
3. 報告書のパスとサマリを返す

## 内部処理

このコマンドは Skill `evaluate-project` を起動するエントリポイント。実装は Skill 側にある。

## オプション

- `--scope`: `rfp_review` / `plan_review` / `code_review` / `all`（デフォルト all）
- `--phases`: 評価フェーズ（カンマ区切り）。例: `P0,P1,X`
- `--severity`: 評価Severity（カンマ区切り）。デフォルト `critical,high`
- `--output`: 報告書の出力パス。デフォルトは `reports/{timestamp}_{project}.md`
- `--recheck`: 既存の評価結果を再評価（探索フェーズをスキップ）

## 引数なし呼び出し時

`/aqg:evaluate` だけ実行された場合は、現在の作業ディレクトリ（`pwd`）を `project_root` として扱う。
