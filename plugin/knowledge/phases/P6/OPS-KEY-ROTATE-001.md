---
id: OPS-KEY-ROTATE-001
title: 運用として鍵ローテーション運用が定常化し、記録・レビューが継続しているか
phase:
- P6
gate: ops/keys
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 鍵保有
---

# OPS-KEY-ROTATE-001: 運用として鍵ローテーション運用が定常化し、記録・レビューが継続しているか

## 観点・確認内容

運用として鍵ローテーション運用が定常化し、記録・レビューが継続しているか

## 適用条件

鍵保有

## OK基準

鍵ローテーションがポリシー通り実施、自動化済み、漏えい時の再発行手順検証済み

## NG基準

ローテ未実施、手動運用、または手順未検証

## 必要証跡

鍵ローテーション記録＋手順書

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
運用関連（典型: docs/ops/, docs/runbook/, ops/, .github/workflows/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [OPS-KEY-ROTATE-001]` を必ず付与

**確認方法・ツール**: KMS + ローテーション自動化

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

（同じGate 'ops/keys' のエントリ）
