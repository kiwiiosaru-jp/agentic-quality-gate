---
id: STAGE-RESTORE-001
title: 本番投入前にバックアップからのリストア演習を実施し、データ整合性とRPO/RTO目標達成を確認したか
phase:
- P4
gate: stage/restore
severity: critical
priority: Must
judge: AI
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 本番投入前
---

# STAGE-RESTORE-001: 本番投入前にバックアップからのリストア演習を実施し、データ整合性とRPO/RTO目標達成を確認したか

## 観点・確認内容

本番投入前にバックアップからのリストア演習を実施し、データ整合性とRPO/RTO目標達成を確認したか

## 適用条件

本番投入前

## OK基準

リストア演習が成功、データ件数・整合性チェックがPass、所要時間がRTO目標値以内

## NG基準

演習未実施、リストア失敗、整合性不一致、またはRTO超過

## 必要証跡

リストア演習レポート（実施日・所要時間・整合性結果）

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
ステージング設定（典型: docs/stage/, env/staging/, 演習レポート）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [STAGE-RESTORE-001]` を必ず付与

**確認方法・ツール**: バックアップツール + 整合性検証スクリプト + 計測

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

（同じGate 'stage/restore' のエントリ）
