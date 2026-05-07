---
id: PERF-SCALE-001
title: 対象システムにおいて水平/垂直スケール戦略が抑止され、計測値が閾値内に収まっているか
phase:
- P1
gate: perf/scaling
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 高負荷想定
---

# PERF-SCALE-001: 対象システムにおいて水平/垂直スケール戦略が抑止され、計測値が閾値内に収まっているか

## 観点・確認内容

対象システムにおいて水平/垂直スケール戦略が抑止され、計測値が閾値内に収まっているか

## 適用条件

高負荷想定

## OK基準

docs/adr/ または docs/design/performance/scaling.md が存在し、必須項目（水平/垂直スケール戦略／スケールトリガ／上限／コスト試算）が全記載され、ARB承認記録が紐付いている

## NG基準

文書不在、または必須項目欠落1件以上、またはARB承認なし

## 必要証跡

スケール戦略書（docs/design/performance/scaling.md）＋ARB承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
性能関連（典型: docs/performance/, benchmarks/, ソースコード）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [PERF-SCALE-001]` を必ず付与

**確認方法・ツール**: [AI自動] 計測ダッシュボード/負荷試験レポート/プロファイル結果の存在・閾値照合（補助ツール: アーキテクチャレビュー + 負荷試験）　／　[Humanレビュー] 性能基準の妥当性レビュー（critical時のサンプリング監査）

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

（同じGate 'perf/scaling' のエントリ）
