# Claude Code Plugin: agentic-quality-gate

> 🚧 **Phase B で内容投入予定**

Claude Code 用のプラグイン本体。**Subagent / Skill / Slash Command / Hook / MCP** を組み合わせて、Agentic Quality Gate を構成します。

## 構成（予定）

```
plugin/
├── .claude-plugin/plugin.json
├── agents/                    # 8 Subagent
│   ├── gate-evaluator.md       # フェーズゲート判定
│   ├── signal-sensor.md        # 外部信号取込
│   ├── reflective-curator.md   # 自社学習からのルール強化
│   ├── feedback-collector.md   # TP/FP/インシデント集約
│   ├── incident-recorder.md    # ポストモーテム支援
│   ├── project-explorer.md     # プロジェクト文脈の把握
│   ├── report-writer.md        # 評価結果の編集出力
│   └── claude-security-adapter.md
├── skills/                    # 6 Skill
│   ├── checklist/              # フェーズ別チェックリスト出力
│   ├── evaluate-project/       # プロジェクト全体評価
│   ├── reflect/                # 内省（インシデントから学習）
│   ├── sense/                  # 外部信号の取込トリガ
│   ├── incident/               # インシデント記録
│   └── claude-security/        # Claude Code 設定の安全性チェック
├── commands/                  # 6 Slash Command
│   ├── checklist.md
│   ├── evaluate.md
│   ├── reflect.md
│   ├── sense.md
│   ├── incident.md
│   └── claude-security.md
├── scripts/                   # 4 Script
│   ├── excel_to_md.py          # master.xlsx → ナレッジ Markdown 変換
│   ├── add_meta_sheets.py
│   ├── add_incidents_sheet.py
│   └── add_claude_security_sheet.py
└── knowledge/                 # 137 ナレッジエントリ + Excel ドライバ
    ├── INDEX.md                # 自動生成
    ├── schema.yaml             # フロントマタースキーマ
    ├── master.xlsx             # Excel 単一ソース（匿名化版）
    ├── phases/
    │   ├── P0-conception/
    │   ├── P1-architecture/
    │   ├── P2-implementation/
    │   ├── P3-test/
    │   ├── P4-staging/
    │   ├── P5-release/
    │   └── P6-operation/
    └── cross-cutting/
        ├── llm/
        ├── compliance/
        ├── governance/
        └── ux/
```

## インストール（予定）

```bash
# Claude Code に Plugin として追加（Phase B 完了後に有効）
claude plugin install <github-url>/plugin
```

## ライセンス

Apache License 2.0（[../LICENSE](../LICENSE)）
