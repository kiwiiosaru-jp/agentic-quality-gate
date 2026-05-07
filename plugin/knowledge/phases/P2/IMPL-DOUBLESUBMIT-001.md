---
id: IMPL-DOUBLESUBMIT-001
title: 実装として二重送信防止（front+back）が遵守され、レビューと自動検査でPassしているか
phase:
- P2
gate: impl/idempotency
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: フォーム
---

# IMPL-DOUBLESUBMIT-001: 実装として二重送信防止（front+back）が遵守され、レビューと自動検査でPassしているか

## 観点・確認内容

実装として二重送信防止（front+back）が遵守され、レビューと自動検査でPassしているか

## 適用条件

フォーム

## OK基準

二重送信防止が front+back 両層に実装、E2E連打テストで防止確認、LLMレビューで漏れ 0件

## NG基準

片側実装のみ、テストFail、またはLLMで漏れ検出

## 必要証跡

二重送信防止実装レビュー＋E2Eテスト結果

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
実装コード（src/, lib/, app/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [IMPL-DOUBLESUBMIT-001]` を必ず付与

**確認方法・ツール**: [AI自動] 実装コードの静的解析/コードレビュー記録/関連テスト結果の照合（補助ツール: 冪等性実装レビュー + 二重実行テスト）　／　[Humanレビュー] 実装レビュー（critical時のサンプリング監査）

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

（同じGate 'impl/idempotency' のエントリ）
