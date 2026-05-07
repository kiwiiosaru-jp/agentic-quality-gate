---
id: GOV-AI-DATAFLOW-001
title: 組織統制としてAIに貼り付ける情報の境界が方針化され、遵守状況が監査可能か
phase:
- cross-cutting
gate: governance/aidata
severity: critical
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: AI利用
---

# GOV-AI-DATAFLOW-001: 組織統制としてAIに貼り付ける情報の境界が方針化され、遵守状況が監査可能か

## 観点・確認内容

組織統制としてAIに貼り付ける情報の境界が方針化され、遵守状況が監査可能か

## 適用条件

AI利用

## OK基準

AIデータ境界ガイド（docs/governance/ai-data-boundary.md）が存在し、DLPルール設定で機密情報のAI送信ブロック稼働、利用ガイドライン教育受講記録100%

## NG基準

ガイド不在、DLPルール未設定、または教育受講漏れ

## 必要証跡

AIデータ境界ガイド＋DLPルール設定＋教育受講記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
統制関連（典型: docs/governance/, docs/policy/, 社内ポータル）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [GOV-AI-DATAFLOW-001]` を必ず付与

**確認方法・ツール**: [AI自動] ITAMレポート/利用ログ/監査証跡の照合（補助ツール: DLPルール + 利用ガイドライン）　／　[Humanレビュー] 統制方針の妥当性レビュー、例外承認

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

（同じGate 'governance/aidata' のエントリ）
