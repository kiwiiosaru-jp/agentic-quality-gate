---
id: SEC-FILE-003
title: 対象コード・設定においてSVG XSS・ファイル名サニタイズに対応する防御機構が実装され、検証されているか
phase:
- P2
gate: security/file
severity: high
priority: Must
judge: AI
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: アップロード機能
---

# SEC-FILE-003: 対象コード・設定においてSVG XSS・ファイル名サニタイズに対応する防御機構が実装され、検証されているか

## 観点・確認内容

対象コード・設定においてSVG XSS・ファイル名サニタイズに対応する防御機構が実装され、検証されているか

## 適用条件

アップロード機能

## OK基準

ファイルアップロード検証（拡張子/MIME/中身/サイズ）全実装、関連テスト全Pass

## NG基準

検証項目欠落 1件以上、または関連テストFail・未実施

## 必要証跡

ファイル検証テストレポート＋実装レビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
セキュリティ設計・実装（典型: docs/security/, .github/dependabot.yml, src/auth/, src/middleware/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [SEC-FILE-003]` を必ず付与

**確認方法・ツール**: magic numberチェッカ + ClamAV + サニタイザ

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

（同じGate 'security/file' のエントリ）
