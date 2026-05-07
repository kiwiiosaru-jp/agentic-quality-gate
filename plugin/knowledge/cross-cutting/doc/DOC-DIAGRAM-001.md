---
id: DOC-DIAGRAM-001
title: ドキュメントとして図（C4等）保守が整備され、最新性が維持されているか
phase:
- cross-cutting
gate: doc/diagram
severity: medium
priority: Should
judge: Both
source: + 追加
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 複雑システム
---

# DOC-DIAGRAM-001: ドキュメントとして図（C4等）保守が整備され、最新性が維持されているか

## 観点・確認内容

ドキュメントとして図（C4等）保守が整備され、最新性が維持されているか

## 適用条件

複雑システム

## OK基準

アーキテクチャ図（docs/architecture/c4/ または arc42構成）が存在し、最新更新日が3ヶ月以内、コードと整合性あり、レビュー記録あり

## NG基準

図不在、陳腐化、コードと不整合、またはレビュー記録なし

## 必要証跡

アーキテクチャ図（docs/architecture/）＋整合性チェック結果＋レビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
ドキュメント整備状況（典型: docs/, README.md, .github/CODEOWNERS）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [DOC-DIAGRAM-001]` を必ず付与

**確認方法・ツール**: [AI自動] ドキュメント（ADR/README/Runbook/図/API spec）の存在・最新化・必須項目の照合（補助ツール: 図のメンテレビュー (C4 / arc42)）　／　[Humanレビュー] ドキュメント内容の妥当性レビュー・承認

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

（同じGate 'doc/diagram' のエントリ）
