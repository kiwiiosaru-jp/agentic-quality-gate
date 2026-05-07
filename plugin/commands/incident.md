---
description: Record an incident, false-positive, near-miss, or observation. Usage- /aqg:incident [type=...] [project=...] [summary=...]
allowed-tools: Read, Write, Bash, Task
---

`/aqg:incident` コマンド：自社経験を記録。

引数: `$ARGUMENTS`

## 使い方

```
/aqg:incident                                                # 対話モード
/aqg:incident type=incident project="ABC社 顧客接点" summary="..."  # 一括指定
/aqg:incident type=false-positive project="..." summary="..."
```

## 動作

`incident` skill を起動。`incident-recorder` agent が master.xlsx の incidents シートに記録。

## オプション

- `type`: incident / false-positive / near-miss / observation
- `project`: プロジェクト名
- `summary`: 1行要約
- `severity`: critical / high / medium / low（不在ならmedium）
- `evidence`: ファイル/PR/Issue/ログURL（任意）

引数なしの場合は対話モードで順次収集。
