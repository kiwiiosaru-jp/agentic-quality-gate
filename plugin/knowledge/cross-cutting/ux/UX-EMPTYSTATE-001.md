---
id: UX-EMPTYSTATE-001
title: UI/UXとして空状態・エラー状態のUXが満たされ、自動・手動テストで確認されているか
phase:
- cross-cutting
gate: ux/state
severity: medium
priority: Should
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: Web系UI
---

# UX-EMPTYSTATE-001: UI/UXとして空状態・エラー状態のUXが満たされ、自動・手動テストで確認されているか

## 観点・確認内容

UI/UXとして空状態・エラー状態のUXが満たされ、自動・手動テストで確認されているか

## 適用条件

Web系UI

## OK基準

UXガイドライン（docs/ux/state-design.md）に空状態・エラー状態の設計指針が記載、対象画面すべてで実装、ユーザビリティテストでPass

## NG基準

ガイドライン不在、未実装画面1件以上、またはユーザビリティテストFail

## 必要証跡

UXガイドライン＋実装レビュー記録＋ユーザビリティテスト結果

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
UX設計（典型: docs/ux/, design-system/, src/components/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [UX-EMPTYSTATE-001]` を必ず付与

**確認方法・ツール**: [AI自動] axe/Lighthouseレポート/ユーザビリティテスト記録の照合（補助ツール: UXレビュー + ユーザビリティテスト）　／　[Humanレビュー] UX判断の妥当性レビュー、ユーザビリティ承認

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

（同じGate 'ux/state' のエントリ）
