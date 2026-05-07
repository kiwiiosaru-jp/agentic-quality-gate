---
id: COST-ESTIMATE-001
title: 概算コスト試算（クラウド/LLM/データ）が事前に試算され、ハードリミット・アラートが設定されているか
phase:
- P0
gate: cost/estimation
severity: high
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 従量課金利用時
---

# COST-ESTIMATE-001: 概算コスト試算（クラウド/LLM/データ）が事前に試算され、ハードリミット・アラートが設定されているか

## 観点・確認内容

概算コスト試算（クラウド/LLM/データ）が事前に試算され、ハードリミット・アラートが設定されているか

## 適用条件

従量課金利用時

## OK基準

docs/cost/estimate.xlsx（または .md）が存在し、必須項目（月次コスト試算／リソース別内訳／スケール仮説（DAU/QPS/データ量）／アラート閾値／レビュー日）が全記載され、PMO承認記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、またはPMO承認なし

## 必要証跡

コスト試算書（docs/cost/estimate.xlsx）＋PMO承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
コスト試算（典型: docs/cost/, docs/finops/, スプレッドシート, README内予算記載）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [COST-ESTIMATE-001]` を必ず付与

**確認方法・ツール**: [AI自動] コスト試算スプレッドシート、予算アラート設定、ハードリミット設定の存在・閾値の照合（補助ツール: コスト試算スプレッドシート + LLM試算 + レビュー）　／　[Humanレビュー] コスト試算の妥当性確認・経営承認

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

（同じGate 'cost/estimation' のエントリ）
