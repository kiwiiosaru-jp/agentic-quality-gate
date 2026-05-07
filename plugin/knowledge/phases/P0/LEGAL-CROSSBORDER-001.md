---
id: LEGAL-CROSSBORDER-001
title: 対象システムにおいて、越境データ移転（GDPR/米輸出規制/中国データ三法）が法務承認を受け文書化されているか
phase:
- P0
gate: legal/crossborder
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 海外利用者
---

# LEGAL-CROSSBORDER-001: 対象システムにおいて、越境データ移転（GDPR/米輸出規制/中国データ三法）が法務承認を受け文書化されているか

## 観点・確認内容

対象システムにおいて、越境データ移転（GDPR/米輸出規制/中国データ三法）が法務承認を受け文書化されているか

## 適用条件

海外利用者

## OK基準

docs/legal/cross-border-transfer.md が存在し、必須項目（移転元/移転先国／対象データ種別／法令適合性（GDPR等）／保護措置（SCC/同等措置）／DPO判定）が全記載され、DPO承認記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、またはDPO承認なし

## 必要証跡

越境データ移転判定書（docs/legal/cross-border-transfer.md）＋DPO承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
法務関連文書（典型: docs/legal/, docs/compliance/, contracts/, 社内文書管理リンク）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LEGAL-CROSSBORDER-001]` を必ず付与

**確認方法・ツール**: [AI自動] 法務関連文書（契約書/利用規約写し/法務承認記録）の存在、必須項目の網羅性、承認記録のリンク有無（補助ツール: 法務レビュー + データフロー図検証）　／　[Humanレビュー] 法務判断・解釈の最終承認（前例なき領域・業法判断）

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

（同じGate 'legal/crossborder' のエントリ）
