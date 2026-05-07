---
id: UX-A11Y-KEYBOARD-001
title: UI/UXとしてキーボード操作完全性が満たされ、自動・手動テストで確認されているか
phase:
- cross-cutting
gate: ux/a11y
severity: medium
priority: Should
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: Web系UI
---

# UX-A11Y-KEYBOARD-001: UI/UXとしてキーボード操作完全性が満たされ、自動・手動テストで確認されているか

## 観点・確認内容

UI/UXとしてキーボード操作完全性が満たされ、自動・手動テストで確認されているか

## 適用条件

Web系UI

## OK基準

axe/Lighthouse自動チェックで違反 0件、手動アクセシビリティテスト Pass

## NG基準

自動チェックで違反1件以上、または手動テスト未実施

## 必要証跡

axe/Lighthouseレポート＋手動テスト記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
UX設計（典型: docs/ux/, design-system/, src/components/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [UX-A11Y-KEYBOARD-001]` を必ず付与

**確認方法・ツール**: axe / Lighthouse / Pa11y + 手動アクセシビリティテスト

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

（同じGate 'ux/a11y' のエントリ）
