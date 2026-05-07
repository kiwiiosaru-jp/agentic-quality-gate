---
name: ADR作成
description: Michael Nygard 形式の Architecture Decision Record を日本語で作成するスキル。Type 1 決定 (不可逆) のみを記録し、代替案と根拠を残す。
---

# ADR (Architecture Decision Record) 作成

## ADR とは

アーキテクチャ上の**重要な決定**を記録する短い文書。2011 年に Michael Nygard が提唱した形式が事実上の標準。

ADR の意義:

- 「なぜそう決めたのか」の歴史を残す (数年後に自分/他人が振り返れる)
- 代替案の検討が抜けていないかのチェックリストになる
- 新規参画者のオンボーディング資料になる
- Type 1 決定 (不可逆) の慎重な検討を強制する

## ADR を書くべきタイミング

以下の決定は **必ず ADR を書く**:

- フレームワーク/ライブラリの主要選定 (DWH, 認証基盤, ELT ツール, BI)
- データベース選定・スキーマ設計方針 (ID体系, パーティション戦略)
- API 設計方針 (REST/gRPC, 認証方式, バージョニング)
- コスト影響が大きい選択 (クラウドリージョン, インスタンスファミリ)
- 不可逆決定 (Type 1 判定された決定)
- 複数選択肢を真剣に比較した決定
- 論争があって決着した決定 (議論の経緯を残す意義)

以下は ADR 不要:

- 命名規則やコーディングスタイル (別途 Style Guide で)
- ライブラリのパッチバージョン更新
- その場限りの一時的な実装選択
- 可逆 (Type 2) で自明な決定

## 標準構造 (Michael Nygard 形式)

```markdown
# ADR-NNNN: 決定タイトル

- 状態 (Status): 提案中 / 承認 / 非推奨 / 置換
- 日付 (Date): YYYY-MM-DD
- 関係者 (Deciders): 名前 / 役割

## コンテキスト (Context)

この決定が必要になった背景。何が問題で、なぜ決定が必要か。
制約条件 (予算、期限、スキル、規制) も記述。

## 決定 (Decision)

何を決めたか。最終的な選択肢を 1 つだけ書く。
選定理由の要点を箇条書きで。

## 代替案 (Alternatives Considered)

検討した選択肢を最低 3 つ列挙。各選択肢について:
- 概要
- 長所
- 短所
- なぜ採用しなかったか

## 結果 (Consequences)

この決定の結果:
- ポジティブな結果
- ネガティブな結果
- 新たに発生するリスク
- 可逆性 (Type 1 / Type 2)
- フォローアップが必要な事項

## 参考 (References)

- 関連ADR (ADR-XXXX への参照)
- 関連ユースケース (UC-XXX)
- 関連リスク (R-XXX)
- 外部文献・ベンチマーク資料
```

## 状態遷移

```
提案中 (Proposed)
   ↓  レビュー・議論・承認
承認 (Accepted)
   ↓  後日の再検討
非推奨 (Deprecated)    置換 (Superseded by ADR-YYYY)
```

- **提案中**: ドラフト。ステークホルダーレビュー中
- **承認**: 決定確定。以降この決定に従う
- **非推奨**: 決定が古くなったが、代替がまだ決まっていない
- **置換**: より新しい ADR で置き換えられた (その ADR への参照を記載)

## 配置規約

```
docs/
└── adr/
    ├── 0001-dwh-platform-selection.md
    ├── 0002-data-transformation-tool.md
    ├── 0003-llm-platform-selection.md
    └── index.md   ← 一覧表
```

ファイル名は `NNNN-decision-title.md` (連番 4 桁 + スラッグ)。
`index.md` には全 ADR の状態・タイトル・日付の一覧表を維持する。

## 例示 1: ADR-0001 ストレージ選定

```markdown
# ADR-0001: データウェアハウスプラットフォームの選定

- 状態: 承認
- 日付: 2026-05-15
- 関係者: 山田 (PM) / 鈴木 (データアーキテクト) / 佐藤 (情シス部長)

## コンテキスト
当社は既存の Excel ベース分析から脱却し、全社共通のデータ分析基盤を構築する。
主要要件:
- AWS 上で動作 (全社クラウド方針)
- データ量: 現状 5TB、3年後 50TB 想定
- クエリ: 業務時間帯に 50 名が同時利用
- 予算: クラウドコスト 年間 3000 万円以内
- 既存スキル: Python/SQL が中心、Spark 経験者は 2 名

## 決定
Databricks (AWS) を採用する。
- Lakehouse アーキで Silver/Gold を Delta Lake 上に構築
- dbt-core を変換層に採用
- Unity Catalog で統合ガバナンス

## 代替案

### A. Snowflake on AWS
- 長所: SQL中心で学習コスト低、クエリ性能良
- 短所: コンピュートコストが高い、ML ワークロードと分離が必要
- 却下理由: 将来の LLM/ML 統合で別基盤が必要になる

### B. BigQuery (GCP)
- 長所: 運用シンプル、コスト明快
- 短所: AWS に限定する方針と不整合
- 却下理由: 全社クラウド方針違反

### C. Redshift
- 長所: AWS ネイティブで統合容易
- 短所: Spark/ML 統合が弱い、スケール時の性能劣化
- 却下理由: 3 年後 50TB での性能懸念

## 結果
- ポジティブ: Lakehouse で BI と ML の両方を 1 基盤で賄える、dbt 標準対応
- ネガティブ: Notebook 文化への業務部門の抵抗が予想される
- 新規リスク: R007 (Databricks コスト超過), R012 (Unity Catalog の運用スキル不足)
- 可逆性: **Type 1** (乗換コスト数千万、データ移行数ヶ月)
- フォローアップ: ハンズオン教育計画を矢羽⑥に組み込む

## 参考
- UC-001, UC-003, UC-007
- R007, R012
- 候補ベンダーのベンチマーク資料 (社内検証日)
```

## 例示 2: ADR-0002 データ変換層ツール選定

```markdown
# ADR-0002: データ変換層ツールの選定

- 状態: 承認
- 日付: 2026-05-20
- 関係者: 鈴木 (データアーキテクト) / 高橋 (データエンジニアリード)

## コンテキスト
Silver/Gold 層の変換ロジックを管理するツールを選定する。
- チームの SQL スキル: 中〜上級
- Python スキル: 中級
- データ品質テストの自動化必須
- 変換系譜 (lineage) の可視化必要
- ADR-0001 で Databricks 採用済み

## 決定
dbt-core を採用する。Databricks の Workflows から起動。

## 代替案

### A. 素の SQL + Airflow
- 長所: 柔軟、コスト ゼロ
- 短所: テスト・lineage・文書化が手作り
- 却下理由: 運用コストが高い

### B. Databricks DLT (Delta Live Tables)
- 長所: Databricks ネイティブ、性能良
- 短所: Databricks ロックイン、dbt ほど成熟していない
- 却下理由: ベンダーロックインを避けたい

### C. Dataform
- 長所: SQL First
- 短所: GCP 連携に最適化されている
- 却下理由: AWS/Databricks 構成と親和性低い

## 結果
- ポジティブ: dbt エコシステム (packages, community) が使える
- ネガティブ: dbt 経験者の採用が難しい市場
- 新規リスク: R011 (dbt 経験者確保困難)
- 可逆性: **Type 2 寄り** (SQL 自体はポータブルなので書き換え可能)
- フォローアップ: 採用計画、内製教育プログラム設計

## 参考
- ADR-0001
- UC-003, UC-008
- R011
```

## 例示 3: ADR-0003 LLM プラットフォーム選定

```markdown
# ADR-0003: LLM プラットフォームの選定

- 状態: 承認
- 日付: 2026-05-25
- 関係者: 佐藤 (CTO) / 鈴木 (データアーキテクト) / 渡辺 (セキュリティ)

## コンテキスト
テキストデータ (顧客問合せログ, 商品説明) の分類・要約に LLM を利用する。
- データには PII が含まれるケースあり
- 月間処理量: 100万件
- 応答レイテンシ: バッチで 1h 以内
- 予算: LLM API 月額 200 万円以内
- コンプライアンス: 個人情報保護法準拠

## 決定
Azure OpenAI Service (GPT-4o) を利用。LangChain で抽象化層を挟む。

## 代替案

### A. OpenAI API 直接利用
- 却下理由: データが米国リージョンのみで、PII 取扱がグレー

### B. AWS Bedrock (Claude)
- 長所: AWS ネイティブ、東京リージョン
- 短所: Azure OpenAI と比較してコストが若干高い、社内既存知見少
- 却下理由: コストと LLM 性能の Balance で A を選定

### C. オンプレ LLM (Llama 3)
- 長所: データ外部送信ゼロ
- 短所: GPU インフラ構築に数千万、運用困難
- 却下理由: 年間 1 億円の予算に不整合

## 結果
- ポジティブ: データが Azure リージョン内、既存 Azure AD 認証と統合容易
- ネガティブ: AWS 主軸の構成にマルチクラウド要素が入る
- 新規リスク: R015 (Azure OpenAI コスト超過), R016 (LLM 出力の再現性)
- 可逆性: **Type 2** (LangChain 抽象化で他モデルに切替可能)
- フォローアップ: PII 検知フィルタの実装、月次コストモニタ

## 参考
- ADR-0001
- UC-005 (顧客問合せ自動分類)
- R015, R016, R019 (PII漏洩)
```

## 作成時のチェックリスト

- [ ] タイトルは「NNNN: 決定の内容」の形式
- [ ] 代替案を最低 3 つ検討
- [ ] 各代替案に却下理由を明記
- [ ] 可逆性 (Type 1 / Type 2) を明記
- [ ] 関連リスク ID をリスクレジスタから引用
- [ ] 関連ユースケース ID を引用
- [ ] フォローアップ事項を結果セクションに記載
- [ ] 日付・関係者を明記

## 参考

- Michael Nygard, "Documenting Architecture Decisions", 2011
- joelparkerhenderson/architecture-decision-record: https://github.com/joelparkerhenderson/architecture-decision-record
- affaan-m/everything-claude-code: https://github.com/affaan-m/everything-claude-code
- `templates/ADRテンプレート.md`
