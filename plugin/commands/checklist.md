---
description: Generate human review checklist filtered by phase/severity/tech. Usage- /aqg:checklist [--phase=...] [--severity=...] [--tech=...]
allowed-tools: Read, Glob, Grep, Write
---

`/aqg:checklist` コマンド：人間レビュー用チェックリストを Markdown で生成。

引数: `$ARGUMENTS`

## 使い方

```
/aqg:checklist                                        # 全件
/aqg:checklist --phase=P2                              # P2のみ
/aqg:checklist --severity=critical,high                # 重要度絞り
/aqg:checklist --phase=P2 --tech=web,llm               # 技術スタック適用条件で絞り
/aqg:checklist --output=docs/review-checklist.md       # ファイル出力
/aqg:checklist --format=github_issues                  # GitHub Issue として起票
```

## 動作

`checklist` skill を起動。フィルタを適用してチェックリストを Markdown で出力。

## オプション

- `--phase`: フェーズ絞り（カンマ区切り）。例: `P0,P1,X`
- `--severity`: 重要度絞り（デフォルト `critical,high`）
- `--tech`: 適用条件マッチ用のtech stack（カンマ区切り）。例: `web,llm,rag`
- `--output`: 出力先（指定なければ標準出力）
- `--format`: `markdown`（デフォルト）/ `github_issues`
