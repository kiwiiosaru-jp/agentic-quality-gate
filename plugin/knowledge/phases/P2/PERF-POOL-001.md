---
id: PERF-POOL-001
title: 対象システムにおいてコネクションプール枯渇防止が抑止され、計測値が閾値内に収まっているか
phase:
- P2
gate: perf/pool
severity: medium
priority: Should
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: DB系
---

# PERF-POOL-001: 対象システムにおいてコネクションプール枯渇防止が抑止され、計測値が閾値内に収まっているか

## 観点・確認内容

対象システムにおいてコネクションプール枯渇防止が抑止され、計測値が閾値内に収まっているか

## 適用条件

DB系

## OK基準

負荷試験下でコネクション枯渇エラー0件、APMでプール使用率閾値以下

## NG基準

枯渇エラー1件以上、または使用率閾値超過

## 必要証跡

負荷試験レポート＋APMダッシュボード

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
性能関連（典型: docs/performance/, benchmarks/, ソースコード）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [PERF-POOL-001]` を必ず付与

**確認方法・ツール**: k6/JMeter + APM (Datadog/NewRelic)

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

（同じGate 'perf/pool' のエントリ）
