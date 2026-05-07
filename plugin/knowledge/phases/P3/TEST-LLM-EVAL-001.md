---
id: TEST-LLM-EVAL-001
title: テスト戦略・実装としてLLM出力の回帰評価セットが遵守され、CIで継続実行されているか
phase:
- P3
gate: test/llm
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: LLM利用系
---

# TEST-LLM-EVAL-001: テスト戦略・実装としてLLM出力の回帰評価セットが遵守され、CIで継続実行されているか

## 観点・確認内容

テスト戦略・実装としてLLM出力の回帰評価セットが遵守され、CIで継続実行されているか

## 適用条件

LLM利用系

## OK基準

LLM評価セットで品質メトリクス（Accuracy/F1等）が閾値以上、回帰テストが定期実行

## NG基準

メトリクス閾値未達、または評価セット未整備、または回帰未実施

## 必要証跡

LLM評価レポート＋評価セット

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
テスト関連（典型: tests/, __tests__/, docs/test/, CI設定）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [TEST-LLM-EVAL-001]` を必ず付与

**確認方法・ツール**: LLM-as-judge / Promptfoo / 自前evals

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

（同じGate 'test/llm' のエントリ）
