---
id: COMPLY-SOC2-001
title: SOC 2に必要な要件・統制が満たされ証跡が保管されているか
phase:
- cross-cutting
gate: compliance/soc2
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: SaaS事業
---

# COMPLY-SOC2-001: SOC 2に必要な要件・統制が満たされ証跡が保管されているか

## 観点・確認内容

SOC 2に必要な要件・統制が満たされ証跡が保管されているか

## 適用条件

SaaS事業

## OK基準

SOC2対応マトリクス（docs/compliance/soc2-matrix.md）が存在し、Trust Service Criteria全項目で対応状況が記載、認定監査人レポート（Type1/2）あり

## NG基準

マトリクス不在、対応欠落、または監査人レポートなし

## 必要証跡

SOC2対応マトリクス＋認定監査人レポート

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
規格対応文書（典型: docs/compliance/{規格名}/, docs/audit/, セキュリティレビュー記録）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [COMPLY-SOC2-001]` を必ず付与

**確認方法・ツール**: [AI自動] 対応マトリクス/監査チェックリストの存在、項目網羅、認定/承認記録の照合（補助ツール: SOC2監査チェックリスト + 認定監査人）　／　[Humanレビュー] 規格適合の妥当性レビューと認定監査人/法務承認

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

（同じGate 'compliance/soc2' のエントリ）
