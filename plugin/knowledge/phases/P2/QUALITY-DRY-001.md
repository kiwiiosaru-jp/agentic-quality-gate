---
id: QUALITY-DRY-001
title: コード品質としてDRY 原則（直し漏れ防止）が遵守され、Lint・レビューでPassしているか
phase:
- P2
gate: quality/dry
severity: medium
priority: Should
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全実装
---

# QUALITY-DRY-001: コード品質としてDRY 原則（直し漏れ防止）が遵守され、Lint・レビューでPassしているか

## 観点・確認内容

コード品質としてDRY 原則（直し漏れ防止）が遵守され、Lint・レビューでPassしているか

## 適用条件

全実装

## OK基準

PMD CPD（または同等ツール）の重複検出が閾値（例: 20行未満）以下、コードレビューでDRY違反指摘 0件

## NG基準

重複閾値超、またはレビュー指摘1件以上

## 必要証跡

重複検出レポート＋コードレビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
コード品質（typically: ESLint設定, .editorconfig, ソースコード）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [QUALITY-DRY-001]` を必ず付与

**確認方法・ツール**: [AI自動] Lintレポート/コードレビュー記録/重複検出レポートの照合（補助ツール: コード重複検出 (PMD CPD) + コードレビュー）　／　[Humanレビュー] コード品質方針の妥当性レビュー

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

（同じGate 'quality/dry' のエントリ）
