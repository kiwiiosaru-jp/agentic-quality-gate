---
id: TEST-NAMING-001
title: テスト戦略・実装としてテスト名を仕様書として書くが遵守され、CIで継続実行されているか
phase:
- P3
gate: test/naming
severity: medium
priority: Should
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全テスト
---

# TEST-NAMING-001: テスト戦略・実装としてテスト名を仕様書として書くが遵守され、CIで継続実行されているか

## 観点・確認内容

テスト戦略・実装としてテスト名を仕様書として書くが遵守され、CIで継続実行されているか

## 適用条件

全テスト

## OK基準

テスト命名規約（docs/test/naming.md）が存在し、Linterで違反 0件、コードレビュー承認済み

## NG基準

規約不在、Lint違反1件以上、または未承認

## 必要証跡

テスト命名規約＋Linterレポート＋レビュー記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
テスト関連（典型: tests/, __tests__/, docs/test/, CI設定）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [TEST-NAMING-001]` を必ず付与

**確認方法・ツール**: [AI自動] テストカバレッジレポート/CI実行履歴/フレーキー率メトリクスの照合（補助ツール: テスト名規約 + Lint）　／　[Humanレビュー] テスト戦略の妥当性レビュー（critical時のサンプリング監査）

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

（同じGate 'test/naming' のエントリ）
