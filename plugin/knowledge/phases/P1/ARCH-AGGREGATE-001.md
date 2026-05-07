---
id: ARCH-AGGREGATE-001
title: アーキテクチャ決定として集約境界とトランザクション境界の整合がADRに記録され承認されているか
phase:
- P1
gate: architecture/ddd
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: DDD採用
---

# ARCH-AGGREGATE-001: アーキテクチャ決定として集約境界とトランザクション境界の整合がADRに記録され承認されているか

## 観点・確認内容

アーキテクチャ決定として集約境界とトランザクション境界の整合がADRに記録され承認されているか

## 適用条件

DDD採用

## OK基準

docs/design/aggregates/ または docs/adr/ に集約境界設計が存在し、必須項目（集約一覧／不変条件／Tx境界整合性／責務）が全記載され、設計レビュー承認記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、または設計レビュー承認なし

## 必要証跡

集約設計（docs/design/aggregates/）＋設計レビュー承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
アーキ設計（典型: docs/adr/, docs/architecture/, design/, 図ファイル）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [ARCH-AGGREGATE-001]` を必ず付与

**確認方法・ツール**: [AI自動] ADR文書の存在、必須項目（選定理由/代替案/トレードオフ）、承認記録の照合（補助ツール: DDD設計レビュー + コンテキストマップ）　／　[Humanレビュー] アーキテクチャ判断の妥当性レビュー（ARB会議）

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

（同じGate 'architecture/ddd' のエントリ）
