---
id: IMPL-DEADLOCK-001
title: 実装としてデッドロック防止が遵守され、レビューと自動検査でPassしているか
phase:
- P2
gate: impl/concurrency
severity: medium
priority: Should
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 複数ロック
---

# IMPL-DEADLOCK-001: 実装としてデッドロック防止が遵守され、レビューと自動検査でPassしているか

## 観点・確認内容

実装としてデッドロック防止が遵守され、レビューと自動検査でPassしているか

## 適用条件

複数ロック

## OK基準

ロック取得順序が文書化（docs/dev/lock-order.md）、LLMレビューでデッドロックパターン検出 0件、ストレステストでデッドロック発生 0件

## NG基準

順序文書なし、LLMで検出、またはストレステストで発生

## 必要証跡

ロック順序文書＋LLMコードレビュー結果＋ストレステスト結果

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
実装コード（src/, lib/, app/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [IMPL-DEADLOCK-001]` を必ず付与

**確認方法・ツール**: [AI自動] 実装コードの静的解析/コードレビュー記録/関連テスト結果の照合（補助ツール: 並行性実装レビュー + 競合テスト）　／　[Humanレビュー] 実装レビュー（critical時のサンプリング監査）

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

（同じGate 'impl/concurrency' のエントリ）
