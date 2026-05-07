---
description: Import Claude Security findings into agentic-quality-gate. Usage- /aqg:claude-security [path] [--auto-merge] [--sample]
allowed-tools: Read, Write, Bash, Task
---

`/aqg:claude-security` コマンド：Claude Security findings 取込。

引数: `$ARGUMENTS`

## 使い方

```
/aqg:claude-security findings.csv                          # CSV 取込
/aqg:claude-security findings.md                           # Markdown 取込
/aqg:claude-security --webhook                             # webhook inbox 全取込
/aqg:claude-security --sample                              # サンプルで動作確認 (admin未設定でも可)
/aqg:claude-security findings.csv --auto-merge             # 評価結果と自動統合
```

## 動作

`claude-security` skill を起動。`claude-security-adapter` agent が
- ファイル/payload をパース
- 8カテゴリ脆弱性 → 本仕組みのID にマッピング
- HIGH/MEDIUM/LOW → critical/high/medium にマッピング
- master.xlsx の claude_security_findings シートに行追加
- /tmp/aqg_eval_claude_security.json に統合形式で出力

## オプション

- `--auto-merge`: 取込即時に effectiveness シートも更新
- `--sample`: examples/claude_security_sample.json を使う（admin未設定での動作確認）
- `--webhook`: reports/claude_security_inbox/ ディレクトリの全 JSON を一括取込

## 前提

組織で Claude Security が有効化されている必要あり：
1. claude.ai/admin-settings/claude-code で有効化
2. GitHub App インストール
3. Premium seat 割当

未設定なら `--sample` で本仕組み側の動作だけ確認可能。
