---
id: STAGE-BACKUP-001
title: ステージング環境で自動バックアップ動作確認が実施され、合格判定が記録されているか
phase:
- P4
gate: stage/backup
severity: critical
priority: Must
judge: AI
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 永続化系
---

# STAGE-BACKUP-001: ステージング環境で自動バックアップ動作確認が実施され、合格判定が記録されているか

## 観点・確認内容

ステージング環境で自動バックアップ動作確認が実施され、合格判定が記録されているか

## 適用条件

永続化系

## OK基準

自動バックアップが期間内に成功、整合性検証Pass、世代管理が方針通り

## NG基準

バックアップ失敗、整合性不一致、または世代不足

## 必要証跡

バックアップ実行ログ＋整合性検証レポート

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
ステージング設定（典型: docs/stage/, env/staging/, 演習レポート）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [STAGE-BACKUP-001]` を必ず付与

**確認方法・ツール**: AWS Backup / Velero + 整合性検証

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

（同じGate 'stage/backup' のエントリ）
