---
id: SEC-TENANT-001
title: 対象コード・設定においてマルチテナント分離戦略（DB/schema/RLS）に対応する防御機構が実装され、検証されている
phase:
- P1
gate: security/tenant
severity: critical
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: SaaS/B2B
---

# SEC-TENANT-001: 対象コード・設定においてマルチテナント分離戦略（DB/schema/RLS）に対応する防御機構が実装され、検証されている

## 観点・確認内容

対象コード・設定においてマルチテナント分離戦略（DB/schema/RLS）に対応する防御機構が実装され、検証されているか

## 適用条件

SaaS/B2B

## OK基準

docs/security/tenant-isolation.md が存在し、必須項目（分離戦略（DB/schema/RLS）／クロステナント検証手順／責任分界点）が全記載され、セキュリティ承認記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、またはセキュリティ承認なし

## 必要証跡

テナント分離設計書（docs/security/tenant-isolation.md）＋セキュリティ承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
セキュリティ設計・実装（典型: docs/security/, .github/dependabot.yml, src/auth/, src/middleware/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [SEC-TENANT-001]` を必ず付与

**確認方法・ツール**: [AI自動] コード/設定/CI実行記録の自動スキャンと結果照合（補助ツール: RLSテスト + テナント境界検証スクリプト）　／　[Humanレビュー] AI判定結果のサンプリング監査（誤検知/見落としの確認）

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

（同じGate 'security/tenant' のエントリ）
