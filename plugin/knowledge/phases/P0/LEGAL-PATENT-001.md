---
id: LEGAL-PATENT-001
title: 対象システムにおいて、特許侵害リスク事前調査が法務承認を受け文書化されているか
phase:
- P0
gate: legal/patent
severity: medium
priority: Should
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 新技術領域
---

# LEGAL-PATENT-001: 対象システムにおいて、特許侵害リスク事前調査が法務承認を受け文書化されているか

## 観点・確認内容

対象システムにおいて、特許侵害リスク事前調査が法務承認を受け文書化されているか

## 適用条件

新技術領域

## OK基準

docs/legal/patent-clearance/{technology}.md が存在し、必須項目（対象技術／関連特許リスト／抵触判定／緩和案／判定日）が全記載され、弁理士承認記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、または弁理士承認なし

## 必要証跡

特許侵害リスク判定書（docs/legal/patent-clearance/）＋弁理士承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
法務関連文書（典型: docs/legal/, docs/compliance/, contracts/, 社内文書管理リンク）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LEGAL-PATENT-001]` を必ず付与

**確認方法・ツール**: [AI自動] 法務関連文書（契約書/利用規約写し/法務承認記録）の存在、必須項目の網羅性、承認記録のリンク有無（補助ツール: 特許DBサーチ + 法務レビュー）　／　[Humanレビュー] 法務判断・解釈の最終承認（前例なき領域・業法判断）

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

（同じGate 'legal/patent' のエントリ）
