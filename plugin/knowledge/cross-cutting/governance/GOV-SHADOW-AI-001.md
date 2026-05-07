---
id: GOV-SHADOW-AI-001
title: 組織統制として会社未承認AIツール使用禁止が方針化され、遵守状況が監査可能か
phase:
- cross-cutting
gate: governance/shadow
severity: critical
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全組織
---

# GOV-SHADOW-AI-001: 組織統制として会社未承認AIツール使用禁止が方針化され、遵守状況が監査可能か

## 観点・確認内容

組織統制として会社未承認AIツール使用禁止が方針化され、遵守状況が監査可能か

## 適用条件

全組織

## OK基準

AI利用ポリシー（docs/governance/ai-usage-policy.md）が存在し、承認済みAIツール一覧があり、ITAMで未承認AI利用検出 0件、教育受講記録100%

## NG基準

ポリシー不在、未承認利用1件以上、または教育受講漏れ

## 必要証跡

AI利用ポリシー＋承認ツール一覧＋ITAMレポート＋教育受講記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
統制関連（典型: docs/governance/, docs/policy/, 社内ポータル）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [GOV-SHADOW-AI-001]` を必ず付与

**確認方法・ツール**: [AI自動] ITAMレポート/利用ログ/監査証跡の照合（補助ツール: ITAM (IT資産管理) + 利用ログ監査）　／　[Humanレビュー] 統制方針の妥当性レビュー、例外承認

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

（同じGate 'governance/shadow' のエントリ）
