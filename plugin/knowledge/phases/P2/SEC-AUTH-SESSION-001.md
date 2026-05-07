---
id: SEC-AUTH-SESSION-001
title: 対象コード・設定においてセッション管理・固定攻撃対策に対応する防御機構が実装され、検証されているか
phase:
- P2
gate: security/auth
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 認証実装
---

# SEC-AUTH-SESSION-001: 対象コード・設定においてセッション管理・固定攻撃対策に対応する防御機構が実装され、検証されているか

## 観点・確認内容

対象コード・設定においてセッション管理・固定攻撃対策に対応する防御機構が実装され、検証されているか

## 適用条件

認証実装

## OK基準

認証実装が規約準拠で実装され、認証関連テスト全Pass

## NG基準

規約逸脱が1件以上、または認証テスト未実施・Fail

## 必要証跡

認証テストレポート＋静的解析結果＋実装レビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
セキュリティ設計・実装（典型: docs/security/, .github/dependabot.yml, src/auth/, src/middleware/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [SEC-AUTH-SESSION-001]` を必ず付与

**確認方法・ツール**: argon2 ライブラリ + カスタム静的解析 + 認証テスト

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

（同じGate 'security/auth' のエントリ）
