---
id: LLM-COST-RUNAWAY-001
title: LLM/エージェント運用においてLLM呼出コスト暴走防止（ハードリミット）に対する防御・検証機構が機能しているか
phase:
- cross-cutting
gate: llm/cost
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: LLM利用
---

# LLM-COST-RUNAWAY-001: LLM/エージェント運用においてLLM呼出コスト暴走防止（ハードリミット）に対する防御・検証機構が機能しているか

## 観点・確認内容

LLM/エージェント運用においてLLM呼出コスト暴走防止（ハードリミット）に対する防御・検証機構が機能しているか

## 適用条件

LLM利用

## OK基準

LLM Provider上限設定済み、ハードリミット動作確認、月次予算アラート設定

## NG基準

上限なし、リミット動作未確認、またはアラートなし

## 必要証跡

Provider設定＋アラート発火記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
LLM運用（典型: docs/llm/, evals/, prompts/, llm-config.yaml）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LLM-COST-RUNAWAY-001]` を必ず付与

**確認方法・ツール**: LLM Provider 上限設定 + 監視ダッシュボード

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

（同じGate 'llm/cost' のエントリ）
