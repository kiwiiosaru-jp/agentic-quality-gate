---
id: IMPL-TX-001
title: 実装としてトランザクション境界実装が遵守され、レビューと自動検査でPassしているか
phase:
- P2
gate: impl/transaction
severity: critical
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 永続化処理
---

# IMPL-TX-001: 実装としてトランザクション境界実装が遵守され、レビューと自動検査でPassしているか

## 観点・確認内容

実装としてトランザクション境界実装が遵守され、レビューと自動検査でPassしているか

## 適用条件

永続化処理

## OK基準

Tx境界実装が設計書通り（docs/design/data/transaction-boundary.md と整合）、LLMコードレビューで境界逸脱検出 0件、結合テストで全シナリオPass

## NG基準

設計と実装が不整合、または境界逸脱1件以上、または結合テストFail

## 必要証跡

Tx境界設計書＋LLMコードレビュー結果＋結合テスト結果

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
実装コード（src/, lib/, app/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [IMPL-TX-001]` を必ず付与

**確認方法・ツール**: [AI自動] 実装コードの静的解析/コードレビュー記録/関連テスト結果の照合（補助ツール: Tx境界実装レビュー + 結合テスト）　／　[Humanレビュー] 実装レビュー（critical時のサンプリング監査）

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

（同じGate 'impl/transaction' のエントリ）
