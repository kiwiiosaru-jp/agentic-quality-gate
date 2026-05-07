---
id: LLM-LIB-HALLUCIN-001
title: LLM/エージェント運用においてAIが提案する架空ライブラリの検証に対する防御・検証機構が機能しているか
phase:
- cross-cutting
gate: llm/lib
severity: high
priority: Must
judge: AI
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: AI支援開発
---

# LLM-LIB-HALLUCIN-001: LLM/エージェント運用においてAIが提案する架空ライブラリの検証に対する防御・検証機構が機能しているか

## 観点・確認内容

LLM/エージェント運用においてAIが提案する架空ライブラリの検証に対する防御・検証機構が機能しているか

## 適用条件

AI支援開発

## OK基準

AI提案ライブラリの実在検証（公式リポジトリ）実施、SCA合格、メンテ活動確認

## NG基準

実在未確認、SCAFail、またはメンテ停止ライブラリ採用

## 必要証跡

ライブラリ検証記録＋SCAレポート

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
LLM運用（典型: docs/llm/, evals/, prompts/, llm-config.yaml）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LLM-LIB-HALLUCIN-001]` を必ず付与

**確認方法・ツール**: ライブラリ実在検証スクリプト + LLMダブルチェック

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

（同じGate 'llm/lib' のエントリ）
