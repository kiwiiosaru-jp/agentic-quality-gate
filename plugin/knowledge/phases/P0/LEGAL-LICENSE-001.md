---
id: LEGAL-LICENSE-001
title: 対象システムにおいて、OSSライセンス（GPL/AGPL汚染、商用可否）が法務承認を受け文書化されているか
phase:
- P0
gate: legal/license
severity: high
priority: Must
judge: AI
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: OSS利用
---

# LEGAL-LICENSE-001: 対象システムにおいて、OSSライセンス（GPL/AGPL汚染、商用可否）が法務承認を受け文書化されているか

## 観点・確認内容

対象システムにおいて、OSSライセンス（GPL/AGPL汚染、商用可否）が法務承認を受け文書化されているか

## 適用条件

OSS利用

## OK基準

対象テーマの法務レビュー完了、判断記録あり、関連文書が最新化

## NG基準

法務レビュー未実施、判断記録欠落、または文書陳腐化

## 必要証跡

法務レビュー記録＋関連文書（社内文書管理）

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
法務関連文書（典型: docs/legal/, docs/compliance/, contracts/, 社内文書管理リンク）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LEGAL-LICENSE-001]` を必ず付与

**確認方法・ツール**: FOSSA / ScanCode / 法務レビュー

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

（同じGate 'legal/license' のエントリ）
