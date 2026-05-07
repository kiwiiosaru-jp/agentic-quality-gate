---
id: PERF-INDEX-001
title: 対象システムにおいてインデックス設計と効果計測が抑止され、計測値が閾値内に収まっているか
phase:
- P2
gate: perf/index
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: DB系
---

# PERF-INDEX-001: 対象システムにおいてインデックス設計と効果計測が抑止され、計測値が閾値内に収まっているか

## 観点・確認内容

対象システムにおいてインデックス設計と効果計測が抑止され、計測値が閾値内に収まっているか

## 適用条件

DB系

## OK基準

主要クエリにEXPLAIN ANALYZE実施、全Index Scan利用、APMでスロークエリ閾値以下

## NG基準

Seq Scan発生のクエリ1件以上、またはAPMでスロークエリ閾値超

## 必要証跡

EXPLAIN結果＋APMダッシュボードURL

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
性能関連（典型: docs/performance/, benchmarks/, ソースコード）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [PERF-INDEX-001]` を必ず付与

**確認方法・ツール**: EXPLAIN ANALYZE + APMスロークエリ計測

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

（同じGate 'perf/index' のエントリ）
