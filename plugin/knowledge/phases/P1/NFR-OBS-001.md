---
id: NFR-OBS-001
title: 非機能要件として可観測性方針（log/metric/trace 三本柱）が合意・文書化され、計測手段が用意されているか
phase:
- P1
gate: nfr/observability
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 本番運用前
---

# NFR-OBS-001: 非機能要件として可観測性方針（log/metric/trace 三本柱）が合意・文書化され、計測手段が用意されているか

## 観点・確認内容

非機能要件として可観測性方針（log/metric/trace 三本柱）が合意・文書化され、計測手段が用意されているか

## 適用条件

本番運用前

## OK基準

docs/nfr/observability.md が存在し、必須項目（log/metric/trace 三本柱方針／ツール／保持期間／分散トレース ID 規約）が全記載され、SRE承認記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、またはSRE承認なし

## 必要証跡

可観測性方針書（docs/nfr/observability.md）＋SRE承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
非機能要件（典型: docs/nfr/, docs/sre/, SLO定義）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [NFR-OBS-001]` を必ず付与

**確認方法・ツール**: [AI自動] SLO/RTO/RPO定義文書、観測ダッシュボード設定、計測手段の存在・閾値の照合（補助ツール: 観測設計レビュー + ダッシュボード整備）　／　[Humanレビュー] 非機能要件の妥当性レビュー・SLO合意承認

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

（同じGate 'nfr/observability' のエントリ）
