---
name: layer-3-architecture
description: 仮説段階のユースケースと前提を、構造化された設計 (ADR, C4, DDD) に落とし込むスキル群。年間1億円規模では戦略的設計に絞り、戦術的詳細は後工程で補う。
---

# Layer 3: アーキテクチャ

## このレイヤーの目的

Layer 2 で仮説化したユースケースと前提を、以下 3 形式で**構造化された設計**に落とし込む:

1. **ADR** (Architecture Decision Record) - 「なぜそう決めたか」の記録
2. **C4 モデル** - 階層的な可視化 (Context → Container → Component → Code)
3. **DDD 戦略的設計** - 境界づけられたコンテキストの同定とコンテキストマップ

青写真段階では**実装詳細に踏み込まない**。粒度は Container レベル、ADR は 3〜5 件、コンテキストは 3〜5 個が目安。

## 推奨実行順序

1. **ADR作成.md** - 重要決定を記録 (Type 1 決定のみ)
2. **C4ダイアグラム.md** - Context図 (C1)、Container図 (C2) を描く
3. **DDDコンテキスト.md** - 境界づけられたコンテキストの同定とマップ作成

## 入力 (前レイヤーから受け取るもの)

- Layer 2 のユースケース一覧
- Layer 2 の前提 (特に Feasibility 軸)
- Layer 2 のステークホルダーマップ
- 非機能要件の概要 (性能・可用性の規模感)

## 出力 (次レイヤーへ渡すもの)

- **ADR** 3〜5 件 (`docs/adr/NNNN-title.md` 形式)
- **C1 システムコンテキスト図** (Mermaid)
- **C2 コンテナ図** (Mermaid)
- **コンテキストマップ** (DDD, ASCII またはMermaid)
- **Type 1 / Type 2 分類表** (ADR 毎)

これらは Layer 4 (要件), Layer 5 (リスク, 特に可逆性分析) の入力となる。

## 年間1億円規模での軽量化方針

- **ADR**: 不可逆 (Type 1) 決定のみ記録。細かいライブラリ選定は記録しない
- **C4**: C1 と C2 までで十分。C3 (Component) は開発フェーズで必要になってから
- **DDD**: 戦略的設計 (境界づけられたコンテキスト、コンテキストマップ) のみ。戦術的設計 (Aggregate, Entity, Value Object) は不要
- **図はMermaid**: 専用ツール (Structurizr, C4-PlantUML 等) は不要

## データ分析基盤の典型アーキテクチャパターン

### パターン A: 3 層 (Bronze/Silver/Gold) メダリオンアーキ
- 最もポピュラー。dbt + DWH の組み合わせで定着
- Container: Ingest / Bronze Layer / Silver Layer / Gold Layer / BI / API

### パターン B: Lambda (バッチ + ストリーム) アーキ
- リアルタイム性が必要な分析 (不正検知, IoT)
- Container: 上記に加え Kafka / Stream Processor を追加

### パターン C: Kappa (ストリーム一本化) アーキ
- イベント駆動が中心のドメイン
- バッチが Stream のリプレイで代替可能

年間 1 億円規模では、通常 **パターン A** で十分。リアルタイム性が本質的に必要な場合のみ B/C を検討。

## 参考

- `licenses/NOTICE.md` - 出典一覧
