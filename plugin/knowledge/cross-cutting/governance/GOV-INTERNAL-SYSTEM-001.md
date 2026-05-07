---
id: GOV-INTERNAL-SYSTEM-001
title: 組織統制として「社内だから安全」幻想の排除（横移動防止）が方針化され、遵守状況が監査可能か
phase:
- cross-cutting
gate: governance/internal
severity: high
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 社内システム
---

# GOV-INTERNAL-SYSTEM-001: 組織統制として「社内だから安全」幻想の排除（横移動防止）が方針化され、遵守状況が監査可能か

## 観点・確認内容

組織統制として「社内だから安全」幻想の排除（横移動防止）が方針化され、遵守状況が監査可能か

## 適用条件

社内システム

## OK基準

ゼロトラスト適用方針（docs/governance/zero-trust-internal.md）が存在し、内部システムも認証認可必須、横移動防止テスト結果あり

## NG基準

方針不在、内部例外が1件以上、または横移動防止未検証

## 必要証跡

ゼロトラスト適用方針＋認証認可設定＋横移動防止テスト結果

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
統制関連（典型: docs/governance/, docs/policy/, 社内ポータル）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [GOV-INTERNAL-SYSTEM-001]` を必ず付与

**確認方法・ツール**: [AI自動] ITAMレポート/利用ログ/監査証跡の照合（補助ツール: ゼロトラストレビュー + ネットワーク分離検証）　／　[Humanレビュー] 統制方針の妥当性レビュー、例外承認

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

（同じGate 'governance/internal' のエントリ）
