---
name: layer-9-operations
description: 運用ドキュメント整備スキル群。ランブック規約、監査証跡設計、シャドーAI禁止、AIデータ境界ガイドを提供。Layer 6 WBS 矢羽⑥ 非機能・運用と連動。
---

# Layer 9: 運用ドキュメント

## 品質ゲート対応

本レイヤーは以下の品質ゲートエントリに対応:
- **DOC-RUNBOOK-001** (high) — ランブック規約.md
- **GOV-AUDIT-TRAIL-001** (high) — 監査証跡設計.md
- **GOV-SHADOW-AI-001** (critical) — シャドーAI禁止ガバナンス.md
- **GOV-AI-DATAFLOW-001** (critical) — AIデータ境界ガイド.md (Layer 8 PII境界_DLP と連携)
- **GOV-INTERNAL-SYSTEM-001** (high) — Layer 5 ゼロトラスト方針 と連携

## このレイヤーの目的

24h365日 99.9% 可用性 + 規制対象システムを運用するための **運用ドキュメント雛形・規約・ガバナンスルール** を整備する。

矢羽⑥ 非機能・運用 (Layer 6) が運用設計の WBS を提供するのに対し、Layer 9 は **運用ドキュメントの中身・規約・テンプレ** を提供する。

## 推奨実行順序

```
1. AIデータ境界ガイド.md   ← AI 利用案件の前提
   ↓
2. シャドーAI禁止ガバナンス.md   ← 全社員ルール
   ↓
3. 監査証跡設計.md   ← セキュリティ基盤
   ↓
4. ランブック規約.md   ← 運用規約 (矢羽⑥ 6.6 と連動)
```

## 入出力

### 入力
- Layer 5 リスクレジスタ (Kill基準, 監視対象)
- Layer 6 矢羽⑥ 非機能・運用 L3 アクティビティ
- Layer 7 データ分類台帳 (機密度別アクセス制御)
- Layer 8 LLM ガバナンス (PII境界, エージェント権限境界)

### 出力
- 運用対象別ランブック群 (Genesys / Databricks / Azure OpenAI / SAP 等)
- 監査証跡設計書 (別ストレージ + WORM + 改ざん防止)
- シャドーAI禁止ポリシー (4,500名規模の社員ルール)
- AIデータ境界ガイド (DLP + 教育受講記録)

## 連携

| Layer | 連携サブスキル | 関係 |
|-------|--------------|------|
| Layer 5 | ゼロトラスト方針.md | 内部例外撤廃で連動 |
| Layer 5 | 暗号化_KMS設計.md | 監査ログ保護 |
| Layer 6 | WBSテンプレート.md (矢羽⑥) | ランブック整備計画 |
| Layer 7 | データ分類台帳.md | 機密度別運用 |
| Layer 8 | PII境界_DLP.md | AIデータ境界統合 |
| Layer 8 | エージェント権限境界書.md | シャドーAI禁止連動 |

## 適用判断

- **規制対象 (個人情報・金融・医療・公共)**: 必須
- **数億円以上・多年次**: 必須
- **24h365日稼働**: 必須
- **小規模・短期**: ランブック規約のみ部分適用

## 参考

- Google SRE Book "Service Best Practices"
- ITIL 4 "Service Operation"
- Microsoft Operations Management Framework
- 既存 Layer 6 WBSテンプレート.md 矢羽⑥構成
