---
description: Run Reflective Curator (4 modes - incidents abstraction / FP analysis / conditional patterns / cross-trend). Usage- /aqg:reflect [--mode=...] [--days=90]
allowed-tools: Read, Write, Bash, Task
---

`/aqg:reflect` コマンド：内省サイクル（Reflective Curator）実行。

引数: `$ARGUMENTS`

## 使い方

```
/aqg:reflect                              # 全モード（all）
/aqg:reflect --mode=incidents-only        # Mode 1 のみ
/aqg:reflect --mode=effectiveness-only    # Mode 2 のみ
/aqg:reflect --mode=conditional-pattern   # Mode 3 のみ
/aqg:reflect --mode=cross-trend           # Mode 4 のみ
/aqg:reflect --days=180                   # 過去180日まで
/aqg:reflect --fp-threshold=20            # FP率閾値変更
```

## 動作

`reflect` skill を起動。`reflective-curator` agent が master.xlsx の incidents/effectiveness/senses から候補ナレッジを生成。

## 4モード

| Mode | 入力 | 出力 |
|---|---|---|
| 1 incidents-only | incidents (raw) | 同種パターンを抽象化、新規/更新candidates |
| 2 effectiveness-only | FP率>閾値のエントリ | OK基準明確化の更新candidates |
| 3 conditional-pattern | reports/ の Conditional 観点 | 観点漏れの新規candidates |
| 4 cross-trend | incidents + senses 相関 | meta-pattern の新規candidates |

## オプション

- `--mode`: `all` (default) / `incidents-only` / `effectiveness-only` / `conditional-pattern` / `cross-trend`
- `--days`: 観測期間（デフォルト 90）
- `--fp-threshold`: FP率閾値（デフォルト 30）
- `--min-pattern-count`: 抽象化の最小件数（デフォルト 2）

## 出力

- master.xlsx 更新（candidates / incidents status / reflections）
- `reports/{ts}_reflective.md`
