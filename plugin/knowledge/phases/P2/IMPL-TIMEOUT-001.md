---
id: IMPL-TIMEOUT-001
title: 実装として全外部呼出にタイムアウト設定が遵守され、レビューと自動検査でPassしているか
phase:
- P2
gate: impl/timeout
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 外部HTTP/DB
---

# IMPL-TIMEOUT-001: 実装として全外部呼出にタイムアウト設定が遵守され、レビューと自動検査でPassしているか

## 観点・確認内容

実装として全外部呼出にタイムアウト設定が遵守され、レビューと自動検査でPassしているか

## 適用条件

外部HTTP/DB

## OK基準

全外部呼出（HTTP/DB/MQ）に静的解析でタイムアウト検出 100%、欠落 0件、関連テスト全Pass

## NG基準

タイムアウト未設定が1件以上、またはテストFail

## 必要証跡

静的解析レポート＋タイムアウト規約（docs/dev/timeout-policy.md）

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
実装コード（src/, lib/, app/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [IMPL-TIMEOUT-001]` を必ず付与

**確認方法・ツール**: Semgrep / カスタム静的解析

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

（同じGate 'impl/timeout' のエントリ）
