---
id: API-ERROR-001
title: API設計としてRFC7807 Problem Details 採用が仕様化され、設計レビュー承認されているか
phase:
- P1
gate: api/error
severity: medium
priority: Should
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: API公開時
---

# API-ERROR-001: API設計としてRFC7807 Problem Details 採用が仕様化され、設計レビュー承認されているか

## 観点・確認内容

API設計としてRFC7807 Problem Details 採用が仕様化され、設計レビュー承認されているか

## 適用条件

API公開時

## OK基準

docs/api/error-format.md または OpenAPI仕様にRFC7807 Problem Details準拠のエラー形式が定義され、必須項目（type/title/status/detail/instance）が全記載され、API設計レビュー承認記録が紐付いている

## NG基準

仕様書不在、または必須項目欠落1件以上、またはRFC7807非準拠、または承認なし

## 必要証跡

エラー応答仕様（docs/api/error-format.md）＋OpenAPI仕様＋設計レビュー承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
API仕様（典型: docs/api/, openapi.yaml, swagger.json, GraphQL schema）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [API-ERROR-001]` を必ず付与

**確認方法・ツール**: [AI自動] API設計文書/OpenAPI仕様/バージョニング規約の存在、項目網羅、承認記録の照合（補助ツール: API設計レビュー + RFC7807 準拠）　／　[Humanレビュー] API設計の妥当性レビュー（設計レビュー会）

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

（同じGate 'api/error' のエントリ）
