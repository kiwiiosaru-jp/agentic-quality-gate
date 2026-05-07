---
id: QUALITY-NAMING-001
title: コード品質として命名規則・自己説明的コードが遵守され、Lint・レビューでPassしているか
phase:
- P2
gate: quality/naming
severity: medium
priority: Should
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全実装
---

# QUALITY-NAMING-001: コード品質として命名規則・自己説明的コードが遵守され、Lint・レビューでPassしているか

## 観点・確認内容

コード品質として命名規則・自己説明的コードが遵守され、Lint・レビューでPassしているか

## 適用条件

全実装

## OK基準

命名規約（docs/dev/naming-conventions.md）が存在し、Linterで違反 0件、コードレビュー承認済み

## NG基準

規約不在、Lint違反1件以上、または未承認

## 必要証跡

命名規約文書＋Linterレポート＋レビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
コード品質（typically: ESLint設定, .editorconfig, ソースコード）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [QUALITY-NAMING-001]` を必ず付与

**確認方法・ツール**: [AI自動] Lintレポート/コードレビュー記録/重複検出レポートの照合（補助ツール: Lint / 命名規約レビュー）　／　[Humanレビュー] コード品質方針の妥当性レビュー

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

（同じGate 'quality/naming' のエントリ）
