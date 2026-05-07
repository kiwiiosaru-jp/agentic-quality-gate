---
id: STAGE-PII-MASK-001
title: ステージング環境でステージング用PII疑似化が実施され、合格判定が記録されているか
phase:
- P4
gate: stage/pii
severity: critical
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: PIIあり
---

# STAGE-PII-MASK-001: ステージング環境でステージング用PII疑似化が実施され、合格判定が記録されているか

## 観点・確認内容

ステージング環境でステージング用PII疑似化が実施され、合格判定が記録されているか

## 適用条件

PIIあり

## OK基準

PII疑似化手順書（docs/stage/pii-masking.md）が存在し、ステージング DB の自動スキャンで原本PII検出 0件、再識別テストで全Pass（再識別不可）

## NG基準

手順書不在、原本PII検出1件以上、または再識別可能

## 必要証跡

PII疑似化手順書＋PIIスキャン結果＋再識別テストレポート

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
ステージング設定（典型: docs/stage/, env/staging/, 演習レポート）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [STAGE-PII-MASK-001]` を必ず付与

**確認方法・ツール**: [AI自動] ステージング演習レポート/構成比較結果/PII疑似化記録の照合（補助ツール: Faker / DPツール + PII検出スキャン）　／　[Humanレビュー] ステージング判定のレビュー・本番投入Go/No-Go判断

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

（同じGate 'stage/pii' のエントリ）
