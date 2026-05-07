---
id: API-STYLE-001
title: API設計としてREST/GraphQL/gRPC選定が仕様化され、設計レビュー承認されているか
phase:
- P1
gate: api/style
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: API公開時
---

# API-STYLE-001: API設計としてREST/GraphQL/gRPC選定が仕様化され、設計レビュー承認されているか

## 観点・確認内容

API設計としてREST/GraphQL/gRPC選定が仕様化され、設計レビュー承認されているか

## 適用条件

API公開時

## OK基準

docs/adr/ にAPIスタイル選定ADR（REST/GraphQL/gRPC）が存在し、必須項目（選定スタイル／選定理由／代替案／互換性方針）が全記載され、ARB承認記録が紐付いている

## NG基準

ADR不在、または必須項目欠落1件以上、またはARB承認なし

## 必要証跡

APIスタイルADR（docs/adr/）＋ARB承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
API仕様（典型: docs/api/, openapi.yaml, swagger.json, GraphQL schema）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [API-STYLE-001]` を必ず付与

**確認方法・ツール**: [AI自動] API設計文書/OpenAPI仕様/バージョニング規約の存在、項目網羅、承認記録の照合（補助ツール: API設計レビュー + ADR）　／　[Humanレビュー] API設計の妥当性レビュー（設計レビュー会）

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

（同じGate 'api/style' のエントリ）
