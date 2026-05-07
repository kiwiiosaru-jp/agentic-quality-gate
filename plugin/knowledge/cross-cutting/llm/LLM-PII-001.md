---
id: LLM-PII-001
title: LLM/エージェント運用においてプロンプト内PII送信制御・学習オプトアウトに対する防御・検証機構が機能しているか
phase:
- cross-cutting
gate: llm/pii
severity: critical
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 個人情報あり
---

# LLM-PII-001: LLM/エージェント運用においてプロンプト内PII送信制御・学習オプトアウトに対する防御・検証機構が機能しているか

## 観点・確認内容

LLM/エージェント運用においてプロンプト内PII送信制御・学習オプトアウトに対する防御・検証機構が機能しているか

## 適用条件

個人情報あり

## OK基準

プロンプトのPIIマスキング実装、学習オプトアウト設定済み、DLPで検出 0件

## NG基準

PII送信が1件以上、オプトアウト未設定、またはDLP未実装

## 必要証跡

DLP設定＋プロンプトレビュー＋オプトアウト設定

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
LLM運用（典型: docs/llm/, evals/, prompts/, llm-config.yaml）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LLM-PII-001]` を必ず付与

**確認方法・ツール**: DLPツール + プロンプトレビュー + 学習オプトアウト確認

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

（同じGate 'llm/pii' のエントリ）
