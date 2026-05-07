---
id: DATA-CONSISTENCY-001
title: データ設計としてACID vs BASE / 結果整合性の許容範囲が文書化され、レビュー承認されているか
phase:
- P1
gate: data/consistency
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 分散システム
---

# DATA-CONSISTENCY-001: データ設計としてACID vs BASE / 結果整合性の許容範囲が文書化され、レビュー承認されているか

## 観点・確認内容

データ設計としてACID vs BASE / 結果整合性の許容範囲が文書化され、レビュー承認されているか

## 適用条件

分散システム

## OK基準

docs/adr/ にACID/BASE選定ADRが存在し、必須項目（データ別整合性レベル／許容範囲／結果整合性の扱い）が全記載され、ARB承認記録が紐付いている

## NG基準

ADR不在、または必須項目欠落1件以上、またはARB承認なし

## 必要証跡

整合性レベルADR（docs/adr/）＋ARB承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
データ設計（典型: docs/design/data/, docs/data/, schemas/, ERD）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [DATA-CONSISTENCY-001]` を必ず付与

**確認方法・ツール**: [AI自動] データ設計文書（ERD/ADR/分類台帳/Tx境界宣言）の存在、必須項目、承認記録の照合（補助ツール: ADRレビュー + 整合性設計）　／　[Humanレビュー] データ設計の妥当性レビュー（設計レビュー会）

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

（同じGate 'data/consistency' のエントリ）
