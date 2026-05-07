---
id: SEC-LIB-003
title: 対象コード・設定においてタイポスクワッティング・依存数最小化に対応する防御機構が実装され、検証されているか
phase:
- P2
gate: security/supply
severity: high
priority: Must
judge: AI
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 依存追加時
---

# SEC-LIB-003: 対象コード・設定においてタイポスクワッティング・依存数最小化に対応する防御機構が実装され、検証されているか

## 観点・確認内容

対象コード・設定においてタイポスクワッティング・依存数最小化に対応する防御機構が実装され、検証されているか

## 適用条件

依存追加時

## OK基準

SCAでCritical脆弱性 0件、lock fileあり、SBOM生成済み、依存最終更新が1年以内

## NG基準

Critical脆弱性 1件以上、またはlock file欠落、またはSBOM未生成、または依存メンテ停止

## 必要証跡

SCAレポート＋SBOM＋依存性レビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
セキュリティ設計・実装（典型: docs/security/, .github/dependabot.yml, src/auth/, src/middleware/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [SEC-LIB-003]` を必ず付与

**確認方法・ツール**: Snyk / Dependabot / Trivy / OSV-Scanner

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

（同じGate 'security/supply' のエントリ）
