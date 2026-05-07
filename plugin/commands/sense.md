---
description: Run external signal sensing only (NVD/PIPC/OWASP). Usage- /aqg:sense [--tech=...] [--days=30]
allowed-tools: WebFetch, WebSearch, Read, Write, Bash, Task
---

`/aqg:sense` コマンド：外部変化サーチを単独実行（評価なし）。

引数: `$ARGUMENTS`

## 使い方

```
/aqg:sense                                # デフォルト（直近30日、全信号源）
/aqg:sense --tech=python,fastapi,azure-openai --days=30
/aqg:sense --sources=nvd,pipc             # 一部の信号源だけ
```

## 動作

`sense` skill を起動。`signal-sensor` agent が Layer 1+2 を実行：
1. NVD / PIPC / OWASP 等から最新変化を取得
2. tech_stack と照合して関連度採点
3. master.xlsx の candidates / senses シートに追記
4. 外部変化レポート（Markdown）を発行

## オプション

- `--tech`: 自社stack（カンマ区切り）。例: `python,fastapi,azure-openai,llm/rag`
- `--days`: 観測期間（デフォルト 30）
- `--sources`: 信号源（デフォルト `nvd,pipc,owasp`）

## 出力

- master.xlsx 更新（candidates, senses シート）
- `reports/{timestamp}_signal_report.md`

人間が candidates を採用判断後、次回 `/aqg:evaluate` で Checklist に自動昇格。
