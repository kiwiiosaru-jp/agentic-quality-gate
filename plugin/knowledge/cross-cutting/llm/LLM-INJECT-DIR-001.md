---
id: LLM-INJECT-DIR-001
title: LLM/エージェント運用においてプロンプトインジェクション（直接）に対する防御・検証機構が機能しているか
phase:
- cross-cutting
gate: llm/injection
severity: critical
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: LLM利用
---

# LLM-INJECT-DIR-001: LLM/エージェント運用においてプロンプトインジェクション（直接）に対する防御・検証機構が機能しているか

## 観点・確認内容

LLM/エージェント運用においてプロンプトインジェクション（直接）に対する防御・検証機構が機能しているか

## 適用条件

LLM利用

## OK基準

プロンプトインジェクション評価セット（docs/llm/prompt-injection-evals.md または評価セットCSV）が存在し、既知ペイロード（OWASP LLM Top10等）で検出率 ≥ 合意閾値（例: 95%）、ガードレール設定（Lakera/Promptfoo等）稼働

## NG基準

評価セット不在、検出率閾値未達、またはガードレール未稼働

## 必要証跡

評価セット＋実行結果＋ガードレール設定

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
LLM運用（典型: docs/llm/, evals/, prompts/, llm-config.yaml）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LLM-INJECT-DIR-001]` を必ず付与

**確認方法・ツール**: [AI自動] 評価セット実行結果/ガードレール設定/ドリフト監視レポートの照合（補助ツール: LLM-as-judge + Lakera / Promptfoo / ガードレール）　／　[Humanレビュー] LLM判定の妥当性レビュー、ガードレール設定の承認

## Humanレビュー観点

判定者が `Both` の場合の人間関与:
- AI のみ: サンプリング監査（誤検知率が10%超なら全件人間レビューに移行）
- Both: AI判定結果のレビュー＋最終承認

## 陳腐化判定基準

- 関連する規格・法令・主要ライブラリの改訂
- 自社で類似のインシデント発生時
- AI判定の False Positive 率 > 30% が3ヶ月続いた場合
- 上記いずれかが発生したら revalidate モードで再検証

## 関連ナレッジ

（同じGate 'llm/injection' のエントリ）
