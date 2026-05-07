---
id: LLM-EVAL-REGR-001
title: LLM/エージェント運用において評価セットによる回帰検知に対する防御・検証機構が機能しているか
phase:
- cross-cutting
gate: llm/eval
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: LLM運用
---

# LLM-EVAL-REGR-001: LLM/エージェント運用において評価セットによる回帰検知に対する防御・検証機構が機能しているか

## 観点・確認内容

LLM/エージェント運用において評価セットによる回帰検知に対する防御・検証機構が機能しているか

## 適用条件

LLM運用

## OK基準

評価セットが整備、CI実行で品質回帰検知、メトリクスがダッシュボード化

## NG基準

評価セット欠落、CI未実行、または回帰検知なし

## 必要証跡

評価セット＋CIレポート＋ダッシュボード

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
LLM運用（典型: docs/llm/, evals/, prompts/, llm-config.yaml）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LLM-EVAL-REGR-001]` を必ず付与

**確認方法・ツール**: Promptfoo / Helm / 自前eval framework

## Humanレビュー観点

判定者が `AI` の場合の人間関与:
- AI のみ: サンプリング監査（誤検知率が10%超なら全件人間レビューに移行）
- Both: AI判定結果のレビュー＋最終承認

## 陳腐化判定基準

- 関連する規格・法令・主要ライブラリの改訂
- 自社で類似のインシデント発生時
- AI判定の False Positive 率 > 30% が3ヶ月続いた場合
- 上記いずれかが発生したら revalidate モードで再検証

## 関連ナレッジ

（同じGate 'llm/eval' のエントリ）
