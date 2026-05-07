---
id: LLM-DRIFT-001
title: LLM/エージェント運用においてモデル更新時の品質ドリフト検知に対する防御・検証機構が機能しているか
phase:
- cross-cutting
gate: llm/drift
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: LLM運用
---

# LLM-DRIFT-001: LLM/エージェント運用においてモデル更新時の品質ドリフト検知に対する防御・検証機構が機能しているか

## 観点・確認内容

LLM/エージェント運用においてモデル更新時の品質ドリフト検知に対する防御・検証機構が機能しているか

## 適用条件

LLM運用

## OK基準

評価セット定期実行、品質メトリクスが閾値以下のドリフト、トレンド監視

## NG基準

定期実行なし、ドリフト閾値超、または監視欠落

## 必要証跡

ドリフト監視ダッシュボード＋評価レポート

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
LLM運用（典型: docs/llm/, evals/, prompts/, llm-config.yaml）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LLM-DRIFT-001]` を必ず付与

**確認方法・ツール**: 評価セット定期実行 + 品質トレンド監視

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

（同じGate 'llm/drift' のエントリ）
