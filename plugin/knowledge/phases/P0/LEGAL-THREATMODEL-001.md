---
id: LEGAL-THREATMODEL-001
title: 対象コード・設定においてSTRIDE/LINDDUN による脅威モデル v0に対応する防御機構が実装され、検証されている
phase:
- P0
gate: security/threatmodel
severity: high
priority: Must
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 設計時
---

# LEGAL-THREATMODEL-001: 対象コード・設定においてSTRIDE/LINDDUN による脅威モデル v0に対応する防御機構が実装され、検証されている

## 観点・確認内容

対象コード・設定においてSTRIDE/LINDDUN による脅威モデル v0に対応する防御機構が実装され、検証されているか

## 適用条件

設計時

## OK基準

docs/security/threat-model.md が存在し、必須項目（攻撃者像／資産一覧／STRIDE/LINDDUN脅威カテゴリ別分析／緩和策／レビュー日）が全記載され、セキュリティチームレビュー記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、またはセキュリティレビュー記録なし

## 必要証跡

脅威モデル文書（docs/security/threat-model.md）＋セキュリティレビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
セキュリティ設計・実装（典型: docs/security/, .github/dependabot.yml, src/auth/, src/middleware/）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [LEGAL-THREATMODEL-001]` を必ず付与

**確認方法・ツール**: [AI自動] コード/設定/CI実行記録の自動スキャンと結果照合（補助ツール: STRIDE/LINDDUNワークショップ + 脅威モデルツール）　／　[Humanレビュー] AI判定結果のサンプリング監査（誤検知/見落としの確認）

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

（同じGate 'security/threatmodel' のエントリ）
