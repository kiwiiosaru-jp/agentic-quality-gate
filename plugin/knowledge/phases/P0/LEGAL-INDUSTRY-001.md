---
id: LEGAL-INDUSTRY-001
title: 対象システムにおいて、業法該当性（金融/医療/保険/運送/不動産/通信）が法務承認を受け文書化されているか
phase:
- P0
gate: legal/industry
severity: critical
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 規制業界
---

# LEGAL-INDUSTRY-001: 対象システムにおいて、業法該当性（金融/医療/保険/運送/不動産/通信）が法務承認を受け文書化されているか

## 観点・確認内容

対象システムにおいて、業法該当性（金融/医療/保険/運送/不動産/通信）が法務承認を受け文書化されているか

## 適用条件

規制業界

## OK基準

docs/legal/industry-applicability.md が存在し、必須項目（想定業界／該当法令一覧／適合性判定／緩和措置／対応責任者）が全記載され、業法専門家承認記録（電子サイン or チケットID）が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、または業法専門家承認なし

## 必要証跡

業法該当性判定書（docs/legal/industry-applicability.md）＋業法専門家承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
法務関連文書（典型: docs/legal/, docs/compliance/, contracts/, 社内文書管理リンク）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LEGAL-INDUSTRY-001]` を必ず付与

**確認方法・ツール**: [AI自動] 法務関連文書（契約書/利用規約写し/法務承認記録）の存在、必須項目の網羅性、承認記録のリンク有無（補助ツール: 業界規制マトリクス + 法務レビュー）　／　[Humanレビュー] 法務判断・解釈の最終承認（前例なき領域・業法判断）

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

（同じGate 'legal/industry' のエントリ）
