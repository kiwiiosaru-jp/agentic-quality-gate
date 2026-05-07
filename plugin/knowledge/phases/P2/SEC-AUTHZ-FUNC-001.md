---
id: SEC-AUTHZ-FUNC-001
title: 対象コード・設定において機能レベル認可（admin系）に対応する防御機構が実装され、検証されているか
phase:
- P2
gate: security/authz
severity: critical
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 管理機能
---

# SEC-AUTHZ-FUNC-001: 対象コード・設定において機能レベル認可（admin系）に対応する防御機構が実装され、検証されているか

## 観点・確認内容

対象コード・設定において機能レベル認可（admin系）に対応する防御機構が実装され、検証されているか

## 適用条件

管理機能

## OK基準

認可検証が全該当箇所に実装、未認可テスト全Pass、コードレビューで漏れなし

## NG基準

認可未実装の箇所が1件以上、または未認可テスト未実施・Fail

## 必要証跡

認可テストレポート＋コードレビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
セキュリティ設計・実装（典型: docs/security/, .github/dependabot.yml, src/auth/, src/middleware/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [SEC-AUTHZ-FUNC-001]` を必ず付与

**確認方法・ツール**: [AI自動] コード/設定/CI実行記録の自動スキャンと結果照合（補助ツール: OPA/Casbin/Cedar + LLMコードレビュー + 認可テスト）　／　[Humanレビュー] AI判定結果のサンプリング監査（誤検知/見落としの確認）

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

（同じGate 'security/authz' のエントリ）
