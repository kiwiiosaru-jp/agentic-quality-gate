---
id: OPS-PATCH-001
title: 運用として依存ライブラリ脆弱性パッチ運用が定常化し、記録・レビューが継続しているか
phase:
- P6
gate: ops/patch
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全システム
---

# OPS-PATCH-001: 運用として依存ライブラリ脆弱性パッチ運用が定常化し、記録・レビューが継続しているか

## 観点・確認内容

運用として依存ライブラリ脆弱性パッチ運用が定常化し、記録・レビューが継続しているか

## 適用条件

全システム

## OK基準

依存パッチ運用が定常化（Renovate/Dependabot等）、Critical脆弱性のパッチが SLA内に適用

## NG基準

パッチ運用なし、またはSLA超過

## 必要証跡

パッチ適用記録＋SCAレポート

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
運用関連（典型: docs/ops/, docs/runbook/, ops/, .github/workflows/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [OPS-PATCH-001]` を必ず付与

**確認方法・ツール**: Renovate / Dependabot + パッチ適用記録

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

（同じGate 'ops/patch' のエントリ）
