---
id: DATA-DELIVERY-001
title: メッセージング設計としてAt-least-once / At-most-once / Exactly-once 選定が文
phase:
- P1
gate: messaging/delivery
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 非同期通信
---

# DATA-DELIVERY-001: メッセージング設計としてAt-least-once / At-most-once / Exactly-once 選定が文

## 観点・確認内容

メッセージング設計としてAt-least-once / At-most-once / Exactly-once 選定が文書化され、設計レビュー承認されているか

## 適用条件

非同期通信

## OK基準

docs/adr/ または docs/design/messaging.md に配信保証レベル選定が存在し、必須項目（保証レベル（At-least/At-most/Exactly-once）／重複排除／DLQ／監視）が全記載され、設計レビュー承認記録が紐付いている

## NG基準

選定文書不在、または必須項目欠落1件以上、または設計レビュー承認なし

## 必要証跡

配信保証ADR（docs/adr/）＋設計レビュー承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
メッセージング設計（典型: docs/design/messaging/, docs/adr/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [DATA-DELIVERY-001]` を必ず付与

**確認方法・ツール**: [AI自動] メッセージング設計文書、配信保証レベルADRの存在・承認記録の照合（補助ツール: メッセージ配信設計レビュー + ADR）　／　[Humanレビュー] メッセージング設計の妥当性レビュー

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

（同じGate 'messaging/delivery' のエントリ）
