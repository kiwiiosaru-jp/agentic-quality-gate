---
id: GIT-COMMIT-001
title: バージョン管理運用として小さく頻繁なコミットが遵守されているか
phase:
- P2
gate: git/commit
severity: medium
priority: Should
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全プロジェクト
---

# GIT-COMMIT-001: バージョン管理運用として小さく頻繁なコミットが遵守されているか

## 観点・確認内容

バージョン管理運用として小さく頻繁なコミットが遵守されているか

## 適用条件

全プロジェクト

## OK基準

コミット粒度ガイド（docs/dev/commit-guide.md）が存在し、Conventional CommitsまたはCommit Lintで違反 0件、平均コミット粒度が閾値以下

## NG基準

ガイド不在、Lint違反1件以上、または粒度逸脱

## 必要証跡

コミットガイド＋Commit Lintレポート＋粒度メトリクス

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
Git運用（.gitignore, .github/, リポジトリ設定）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [GIT-COMMIT-001]` を必ず付与

**確認方法・ツール**: [AI自動] .gitignore設定/secret scan結果/コミット規約Lintの照合（補助ツール: コミット粒度ガイド + コードレビュー）　／　[Humanレビュー] Git運用方針の妥当性レビュー

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

（同じGate 'git/commit' のエントリ）
