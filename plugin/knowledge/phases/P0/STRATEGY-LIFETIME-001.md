---
id: STRATEGY-LIFETIME-001
title: プロジェクトとして「使い捨て or 恒久」宣言と品質投資の整合が明文化され、関係者に共有されているか
phase:
- P0
gate: strategy/lifetime
severity: high
priority: Must
judge: Both
source: ★ 記事
status: active
last_validated: '2026-04-30'
review_frequency: quarterly
applies_when: 全プロジェクト
---

# STRATEGY-LIFETIME-001: プロジェクトとして「使い捨て or 恒久」宣言と品質投資の整合が明文化され、関係者に共有されているか

## 観点・確認内容

プロジェクトとして「使い捨て or 恒久」宣言と品質投資の整合が明文化され、関係者に共有されているか

## 適用条件

全プロジェクト

## OK基準

docs/project/lifetime.md（またはREADME内 'Lifetime' セクション）が存在し、必須項目（想定寿命／使い捨て or 恒久の宣言／品質投資レベル／昇格判断基準）が全記載され、PJオーナー承認記録が紐付いている

## NG基準

ファイル不在、または必須項目欠落1件以上、またはPJオーナー承認なし

## 必要証跡

プロジェクト寿命宣言書（docs/project/lifetime.md または README）＋PJオーナー承認記録

## AI自律確認の手順

**AI agentが探すべき場所のヒント（決め打ちではない）**:
プロジェクト戦略文書（典型: docs/project/, README, kickoff資料）

**AI が実行する確認ステップ**:
1. プロジェクトルートから上記ヒントに沿って `Glob`/`Grep` で関連ファイルを探索
2. ファイル内容を `Read` で確認、必須項目の網羅性を判定
3. 承認記録（PR Approve / 電子サイン / チケットID）の紐付けを確認
4. 上記「OK基準」を満たすか判定し、不足があれば「NG基準」と「不足要素」を引用付きで報告
5. 判定結果は `cited_knowledge: [STRATEGY-LIFETIME-001]` を必ず付与

**確認方法・ツール**: [AI自動] プロジェクト寿命宣言文書、品質投資レビュー記録の存在・承認状態の照合（補助ツール: プロジェクト寿命宣言文書 + 投資レビュー）　／　[Humanレビュー] プロジェクト寿命と品質投資のバランス判断・経営承認

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

（同じGate 'strategy/lifetime' のエントリ）
