---
name: layer-8-llm-governance
description: LLM/AI エージェント特有のガバナンスを設計するスキル群。エージェント権限境界、プロンプトインジェクション評価、ハルシネーション対策、PII境界、出力検証、モデルドリフト検知、評価セット回帰検知を統合。OWASP LLM Top10 と AI ガバナンス監査要件に対応。
---

# Layer 8: LLM ガバナンス

## このレイヤーの目的

LLM (Large Language Model) や AI エージェントを業務プロセスに組み込む際、従来のリスクレジスタや STRIDE では捉えきれない**LLM 固有のガバナンス課題**を体系的に設計する。

具体的には:

- **エージェント権限の境界** : LLM がどこまで自律的に操作してよいか
- **プロンプトインジェクション** : 直接/間接的な指示奪取攻撃への対応
- **ハルシネーション** : 事実に基づかない出力の検出と封じ込め
- **PII 漏洩** : LLM 入出力経路での個人情報保護
- **出力の信頼性** : SQL/コマンド/コードを安全に実行する仕組み
- **モデルドリフト** : ベンダー側の暗黙的更新による品質変化検知
- **評価セット回帰** : CI/CD で自動的に品質劣化を防ぐ仕組み

本レイヤーは品質評価レポート (`/Users/shigeru.abe/agentic-quality-gate/reports/2026-04-30_blueprint_review.md`) で **Fail/Conditional** と判定された LLM ガバナンス系 9 件 (LLM-AGENCY-001、LLM-INJECT-DIR-001、LLM-INJECT-INDIR-001、LLM-OUTPUT-TRUST-001、LLM-PII-001、LLM-HALLUCIN-001、LLM-DRIFT-001、LLM-EVAL-REGR-001、GOV-AI-DATAFLOW-001) を Pass に引き上げるための文書化テンプレ・評価セット・運用設計の集合体である。

## 対応する品質評価ID

| 品質ID | 評価 | 対応ファイル |
|---|---|---|
| LLM-AGENCY-001 | Fail | `エージェント権限境界書.md` |
| LLM-INJECT-DIR-001 | Fail | `プロンプトインジェクション評価.md` |
| LLM-INJECT-INDIR-001 | Fail | `プロンプトインジェクション評価.md` |
| LLM-HALLUCIN-001 | Conditional | `ハルシネーション対策.md` |
| LLM-PII-001 | Fail | `PII境界_DLP.md` |
| LLM-OUTPUT-TRUST-001 | Fail | `LLM出力検証_許可リスト.md` |
| LLM-DRIFT-001 | Conditional | `モデルドリフト検知.md` |
| LLM-EVAL-REGR-001 | Fail | `評価セット_回帰検知.md` |
| GOV-AI-DATAFLOW-001 | Conditional | 全ファイル横断 (本 SKILL.md でフロー定義) |

## いつ使うか

以下のいずれかに該当する案件で**必ず**適用する:

- LLM を業務処理に組み込む (要約、分類、生成、Text2SQL、ボイスボット)
- AI エージェント (Claude Code, Copilot, AutoGen, LangGraph 等) を本番に乗せる
- RAG (Retrieval-Augmented Generation) による問合せ応答を提供する
- 顧客接点 (チャット、電話) で LLM が応対する
- LLM 出力を業務システムに自動連携 (SQL 実行、CRM 更新、メール送信)
- 個人情報や機微情報が LLM プロンプトに到達しうる経路がある

## 推奨実行順序

```
1. エージェント権限境界書.md       (どこまで自律操作を許すかを先に決める)
   ↓
2. PII境界_DLP.md                 (LLM への入出力経路を明示する)
   ↓
3. プロンプトインジェクション評価.md (ペイロード集と評価セットを整備)
   ↓
4. LLM出力検証_許可リスト.md       (出力の Sandbox 実行・許可リスト方式)
   ↓
5. ハルシネーション対策.md          (引用必須化と出典検証層)
   ↓
6. 評価セット_回帰検知.md          (CI/CD 回帰ゲートを構築)
   ↓
7. モデルドリフト検知.md            (運用後のメトリクス監視と再評価)
```

最初に「権限の天井」を決め、次に「入出力境界」を引き、次に「攻撃と漏洩の検知」を仕込み、最後に「継続評価」の仕組みを敷く、という順序を強く推奨する。

## custom/AI駆動開発リスク.md との関係

`custom/AI駆動開発リスク.md` は AI 駆動開発の **13 項目チェックリスト** を提供する。本レイヤーはそのうち以下の項目を**深掘り・運用化**したものである:

| custom/AI駆動開発リスク.md | Layer 8 でのカバー |
|---|---|
| R-AI-07 モデル変更による再現性喪失 | `モデルドリフト検知.md` + `評価セット_回帰検知.md` |
| R-AI-08 テストカバレッジの表面的拡張 | `評価セット_回帰検知.md` (Mutation/敵対的サンプル) |
| R-AI-12 プロンプトインジェクション懸念 | `プロンプトインジェクション評価.md` (OWASP LLM01 ペイロード集) |

新しい論点として:

- **エージェント権限** (LLM01 + LLM06)
- **PII/DLP 境界** (LLM02 + LLM06)
- **出力信頼** (LLM02 + LLM05)
- **ハルシネーション** (LLM09)

を追加で扱う。13 項目チェックリストとは**併用**であり、置き換えではない。

## Layer 5 STRIDE との連携

Layer 5 `脅威モデリング.md` で引いた信頼境界とフロー (F001〜F005) のうち、LLM が関与するフローには本レイヤーを**追加適用**する:

- F005 (LLM API → 変換層) のような PII 含有フロー → `PII境界_DLP.md`
- F002/F003 のような Bronze/Silver 書き込みに LLM が介在するなら → `LLM出力検証_許可リスト.md`
- LLM が外部から取得した文書を要約するフロー → `プロンプトインジェクション評価.md` (間接インジェクション)

STRIDE では情報漏洩 (I) と権限昇格 (E) が特に LLM ガバナンスと強く結びつく。

## 入力 (前レイヤーから受け取るもの)

- Layer 3 の C4 Container 図 (LLM コンポーネントの位置)
- Layer 4 のユースケース (UC-003 ボイスボット、UC-008 LLM 分類、UC-011 Text2SQL、UC-012 報告書生成、UC-013 ロイヤル方針 等)
- Layer 4 の機密度ラベル (PII / 機微 / 一般)
- Layer 5 のリスクレジスタ (R-AI-XX 系)
- Layer 5 の STRIDE 結果 (信頼境界・フローID)

## 出力 (次レイヤーへ渡すもの)

- **エージェント権限境界書** (`エージェント権限境界書.md` の YAML 出力)
- **プロンプトインジェクション評価セット** (`promptfoo` YAML / `Lakera Guard` 設定)
- **PII 検出ルール** (regex / NER / DLP ポリシー)
- **出力許可リスト** (SQL/コマンド/コードの allowlist)
- **ハルシネーション検証ポリシー** (引用必須化ルール)
- **評価セット回帰ゲート** (CI/CD pipeline 設定)
- **ドリフト監視ダッシュボード仕様** (メトリクス + 閾値 + アラート)

これらは Layer 6 (WBS) の運用 Sprint および Layer 1 (経営判断) の継続コスト試算に組み込む。

## データ分析基盤・コールセンターでの典型適用例

| ユースケース | 主要適用ファイル |
|---|---|
| UC-003 ボイスボット (顧客電話応対) | `エージェント権限境界書.md` + `PII境界_DLP.md` + `ハルシネーション対策.md` |
| UC-008 LLM 分類 (問合せ自動カテゴライズ) | `モデルドリフト検知.md` + `評価セット_回帰検知.md` |
| UC-011 Text2SQL (自然言語 → SQL) | `LLM出力検証_許可リスト.md` + `エージェント権限境界書.md` (Read-only 強制) |
| UC-012 報告書生成 (経営層向けレポート) | `ハルシネーション対策.md` + `LLM出力検証_許可リスト.md` |
| UC-013 ロイヤル方針 (FAQ ボット) | `プロンプトインジェクション評価.md` + `ハルシネーション対策.md` |

## OWASP LLM Top10 (2025) との対応

本レイヤーが対応する OWASP LLM Top10 カテゴリ:

| OWASP ID | 名称 | 主担当ファイル |
|---|---|---|
| LLM01 | Prompt Injection | `プロンプトインジェクション評価.md` |
| LLM02 | Sensitive Information Disclosure | `PII境界_DLP.md` |
| LLM03 | Supply Chain | `モデルドリフト検知.md` (ベンダー側変更) |
| LLM04 | Data and Model Poisoning | `評価セット_回帰検知.md` |
| LLM05 | Improper Output Handling | `LLM出力検証_許可リスト.md` |
| LLM06 | Excessive Agency | `エージェント権限境界書.md` |
| LLM07 | System Prompt Leakage | `プロンプトインジェクション評価.md` |
| LLM08 | Vector and Embedding Weaknesses | `プロンプトインジェクション評価.md` (間接) |
| LLM09 | Misinformation | `ハルシネーション対策.md` |
| LLM10 | Unbounded Consumption | (Layer 5 R-AI-11 と Layer 6 SLO) |

## 軽量化方針 (年間 1 億円規模)

全ファイルを完全実装せず、案件特性に応じて以下を**最低ライン**として運用:

- **必須**: `エージェント権限境界書.md`、`PII境界_DLP.md`、`評価セット_回帰検知.md`
- **LLM が外部入力を受ける場合は追加**: `プロンプトインジェクション評価.md`
- **LLM が業務システムを操作する場合は追加**: `LLM出力検証_許可リスト.md`
- **LLM が事実情報を提供する場合は追加**: `ハルシネーション対策.md`
- **モデルバージョン非固定の場合は追加**: `モデルドリフト検知.md`

## 出力フォーマット

各ファイルの出力は以下の 3 階層で管理:

1. **設計書** (Markdown) : 本レイヤー内の各ファイルが該当
2. **設定ファイル** (YAML/JSON) : Promptfoo / Lakera / DLP ポリシー
3. **CI/CD 統合** (GitHub Actions / Azure DevOps) : 自動回帰検知

## pm-blueprint 連携

| 連携先 | 関係 |
|---|---|
| `custom/AI駆動開発リスク.md` | 13 項目チェックリストの**深掘り版**として位置づけ。本レイヤーは R-AI-07/08/12 を運用化 |
| `layer-5-risk/脅威モデリング.md` F2/F3 | LLM が関与するフローに本レイヤーを追加適用。STRIDE の I/E と本レイヤーが強く接続 |
| `layer-4-requirements/NFR.md` | セキュリティ NFR (CIA + プライバシー) と本レイヤーの DLP/許可リストが整合 |
| `layer-6-wbs/` | 運用 Sprint に評価セット運用とドリフト監視を組み込む |

## 品質ゲート連動

本レイヤーは品質評価レポートの以下 ID の Pass 条件を満たすために設計されている:

- LLM-AGENCY-001 → `エージェント権限境界書.md` の YAML 表現で Pass
- LLM-INJECT-DIR-001 / INDIR-001 → `プロンプトインジェクション評価.md` の評価セット & 検出率閾値で Pass
- LLM-HALLUCIN-001 → `ハルシネーション対策.md` の引用必須化 & 検証層で Pass
- LLM-PII-001 → `PII境界_DLP.md` の DLP 検出 & マスキング & オプトアウト契約で Pass
- LLM-OUTPUT-TRUST-001 → `LLM出力検証_許可リスト.md` の Sandbox & allowlist で Pass
- LLM-DRIFT-001 → `モデルドリフト検知.md` のメトリクス & ダッシュボードで Pass
- LLM-EVAL-REGR-001 → `評価セット_回帰検知.md` の CI/CD ゲート & 閾値で Pass
- GOV-AI-DATAFLOW-001 → 本 SKILL.md のフロー定義 + 各ファイルの統合で Pass

## 参考

- OWASP, "OWASP Top 10 for LLM Applications 2025"
- Anthropic, "Claude Constitutional AI Principles"
- NIST AI RMF (AI Risk Management Framework, 2023)
- ISO/IEC 42001 (AI Management System, 2023)
- 個人情報保護委員会「生成 AI サービスの利用に関する注意喚起」(2023-06)
- Microsoft Responsible AI Standard v2 (2022)
- `custom/AI駆動開発リスク.md` (本レイヤーのチェックリスト版)
- `layer-5-risk/脅威モデリング.md` (本レイヤーと STRIDE で連携)
