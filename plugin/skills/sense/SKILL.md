---
name: sense
description: |
  Run external signal sensing only (Layer 1+2 standalone). NVD/PIPC/OWASP の最新変化を取り込み、自社stack関連度を採点して候補を master.xlsx に追加する。評価は実行しない。
  Use when user wants to refresh knowledge from external sources without evaluating a project.
  Examples: "外部変化だけサーチして", "/aqg:sense", "ナレッジを最新化", "CVE/規制の差分だけ見たい".
---

# Skill: sense

外部変化サーチ（Layer 1+2）を **単独で** 実行する。評価対象プロジェクトがない場合や、人間レビュー会の前に最新情報だけ取り込みたい場合に使う。

## 起動条件

- 「外部変化をサーチして」
- 「ナレッジを最新化」
- 「/aqg:sense」
- 「CVE/規制の最新を取り込んで」

## 必要な情報

### 必須

なし。デフォルトで動く。

### オプション

- **`tech_stack`**: 自社stackリスト（例: `["python","fastapi","azure-openai","llm/rag"]`）
  - 指定なしの場合は組織標準（README.md の冒頭等から推定 or 全件 high とみなす）
- **`lookback_days`**: 観測期間（デフォルト 30）
- **`sources`**: 信号源（デフォルト `["nvd","pipc","owasp"]`）

## 実行手順

`signal-sensor` agent を Task ツールで起動する：

```
入力:
  tech_stack: <指定された tech_stack または ["any"]>
  lookback_days: 30
  master_xlsx: $CLAUDE_PLUGIN_ROOT/knowledge/master.xlsx
  project_root: (なし or "n/a")
```

## 出力

`signal-sensor` から以下を受け取り、ユーザーに報告：

```
✅ 外部変化サーチ完了

📊 信号取込
- NVD CVE: N件 (critical: M, high: K)
- PIPC: N件
- OWASP: N件

🎯 自社関連: 合計 M件

🆕 ナレッジ追加候補（master.xlsx の candidates シート）: N件
📝 既存更新候補: M件

レポート: $CLAUDE_PLUGIN_ROOT/reports/{ts}_signal_report.md

次のアクション:
→ master.xlsx の candidates シートで人間レビュー
→ 採用するなら status を 'promoted' に変更
→ 次回 /aqg:evaluate 実行時に Phase 1 で Checklist へ昇格
```

## ユースケース

| シーン | 使い方 |
|---|---|
| 月次レビュー会議の前 | 直近30日の変化を取り込み、レビュー会の議題に |
| 業界事故が報道された | 関連CVEがないか、即座に sense で確認 |
| 評価実行前の事前点検 | 何か大きな変化があったかだけ先に確認 |
| 新規プロジェクト着手時 | 関連tech_stackで sense してナレッジを最新化 |

## 制約

- 評価は実行しない。評価したい場合は `/aqg:evaluate` を使う
- 候補は `status=candidate` で保存される。Checklist への昇格は人間レビュー後に手動 or 次回 evaluate 時に自動

## 関連スキル

- `/aqg:evaluate` — 評価込みのフルパイプライン（Phase 0 として sense 内蔵）
- `/aqg:checklist` — 既存ナレッジから人間レビュー用チェックリスト出力
