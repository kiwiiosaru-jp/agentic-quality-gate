---
id: REL-ONCALL-001
title: リリース運用としてオンコール体制・エスカレーションが準備・検証されているか
phase:
- P5
gate: release/oncall
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 本番運用
---

# REL-ONCALL-001: リリース運用としてオンコール体制・エスカレーションが準備・検証されているか

## 観点・確認内容

リリース運用としてオンコール体制・エスカレーションが準備・検証されているか

## 適用条件

本番運用

## OK基準

オンコール体制設定書（docs/ops/oncall.md）が存在し、PagerDuty等のシフト設定で夜間/休日カバー、エスカレーション経路（L1→L2→L3）が明確

## NG基準

設定書不在、シフトカバー漏れ、またはエスカレーション不明

## 必要証跡

オンコール設定書＋PagerDutyシフトエクスポート＋エスカレーション図

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
リリース手順（典型: docs/release/, RELEASE.md, .github/workflows/release.yml）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [REL-ONCALL-001]` を必ず付与

**確認方法・ツール**: [AI自動] リリース手順書/演習レポート/Feature Flag設定/オンコール体制設定の照合（補助ツール: PagerDuty / Opsgenie + エスカレーション設定）　／　[Humanレビュー] リリースGo/No-Go判断、ロールバック判断、対外公表判断

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

（同じGate 'release/oncall' のエントリ）
