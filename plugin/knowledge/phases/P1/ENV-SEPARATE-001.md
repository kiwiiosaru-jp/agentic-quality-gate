---
id: ENV-SEPARATE-001
title: 環境設計としてdev/stg/prod 環境分離（DB含む）が実装され、構成ドリフトが検知できるか
phase:
- P1
gate: env/separation
severity: critical
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全プロジェクト
---

# ENV-SEPARATE-001: 環境設計としてdev/stg/prod 環境分離（DB含む）が実装され、構成ドリフトが検知できるか

## 観点・確認内容

環境設計としてdev/stg/prod 環境分離（DB含む）が実装され、構成ドリフトが検知できるか

## 適用条件

全プロジェクト

## OK基準

docs/infra/environments.md が存在し、dev/stg/prod のVPC/DB/IAMが完全分離されている記載があり、Terraform/Bicep等で構成ドリフト検知設定済み（CI実行記録あり）

## NG基準

ファイル不在、分離不完全（DB共有等）、またはドリフト検知未稼働

## 必要証跡

環境設計書（docs/infra/environments.md）＋IaCコード＋ドリフト検知CI実行記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
環境設計（典型: docs/infra/, terraform/, k8s/, env/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [ENV-SEPARATE-001]` を必ず付与

**確認方法・ツール**: [AI自動] 環境分離アーキテクチャ図、ドリフト検知設定、データフロー方針文書の存在・整合性照合（補助ツール: 環境分離アーキテクチャレビュー + 構成ドリフト検知）　／　[Humanレビュー] 環境設計の妥当性レビュー

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

（同じGate 'env/separation' のエントリ）
