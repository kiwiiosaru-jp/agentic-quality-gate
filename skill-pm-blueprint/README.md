# pm-blueprint

> プロジェクト開始前の **青写真作成** を支援する Claude Code 統合 PM スキル。
> RFP / 要件概要から、計画書類一式（10〜30 文書規模）を**仮説ベースで一気通貫に描ききる**。

姉妹プロジェクトの [agentic-quality-gate Plugin](../plugin/) と組み合わせると、本 Skill が生成した計画書を **品質ゲートエージェント** が自動評価し、改善ループが回ります（解説：[Qiita 記事](https://qiita.com/) §6 参照）。

## 目的

限られた情報から、以下を仮説ベースで一気通貫に描く：

- 経営層視点の投資判断
- ユースケースと前提
- 最適アーキテクチャ
- 機能・非機能要件
- リスクと Kill 基準
- 矢羽パターンのハイブリッド型 WBS
- 法務・コンプライアンス対応マトリクス
- LLM ガバナンス（PII 境界 / 出力検証 / 評価セット）
- 運用設計（監査証跡 / シャドー AI 禁止 / ランブック）

## 9 レイヤー構成

| レイヤー | ディレクトリ | 役割 |
|---|---|---|
| **Layer 1** Executive | `layer-1-executive/` | 投資判断・ピボット・撤退判断 |
| **Layer 2** Hypothesis | `layer-2-hypothesis/` | 前提抽出・ディスカバリー・ロードマップ |
| **Layer 3** Architecture | `layer-3-architecture/` | ADR・C4・DDD・データ設計・環境分離 IaC |
| **Layer 4** Requirements | `layer-4-requirements/` | ユースケース・SMART-NFR・EARS・SLI/SLO・RTO/RPO |
| **Layer 5** Risk | `layer-5-risk/` | 事前検死・リスクレジスタ・脅威モデリング・LINDDUN・暗号化/KMS・認可方式・ゼロトラスト |
| **Layer 6** Execution | `layer-6-execution/` | 矢羽パターン WBS・トレーサビリティ |
| **Layer 7** Legal & Compliance | `layer-7-legal-compliance/` | 個人情報取扱台帳・個情法対応マトリクス・業法該当性判定書・データ分類台帳・OSS ライセンス管理・越境データ移転書・AI 生成物著作権ポリシー・データ保持/削除戦略 |
| **Layer 8** LLM Governance | `layer-8-llm-governance/` | PII 境界 DLP・LLM 出力検証許可リスト・モデルドリフト検知・ハルシネーション対策・評価セット回帰検知・エージェント権限境界書・プロンプトインジェクション評価 |
| **Layer 9** Operations | `layer-9-operations/` | AI データ境界ガイド・監査証跡設計・シャドー AI 禁止ガバナンス・ランブック規約 |

加えて、独自拡張：

- `custom/` — 統合オーケストレーター、AI 駆動開発リスク、日本型 PM コンテキスト、品質ゲート連携
- `templates/` — プロジェクト憲章 / リスクレジスタ YAML / ADR / ユースケース / WBS 雛形 / OpenAPI 仕様書 / SLO 定義書 / DR 設計書 / ERD 仕様書 / Tx 境界宣言 / PII 取扱台帳 / 個情法対応マトリクス / データ分類台帳 / 業法該当性判定書 / 越境データ移転書 / エージェント権限境界書 / ランブック規約 等
- `examples/` — 架空プロジェクト「株式会社サンプル商事 顧客行動分析基盤」の適用例

## 対応規模

| 項目 | 想定値 |
|---|---|
| 年間予算 | 1 億円〜1.5 億円 |
| チーム規模 | 3〜8 名 |
| 期間 | 6〜18 ヶ月 |
| WBS 構造 | 矢羽 (L2) / アクティビティ (L3) / タスク (L4) |
| 開発手法 | ハイブリッド（全体ウォーターフォール + 並列 Sprint） |
| 対象ドメイン | データ分析基盤 / SaaS / 業務システム全般（応用可） |

## インストール

詳細は [INSTALL.md](INSTALL.md) を参照。

```bash
# Claude Code Skill としてインストール（例：~/.claude/skills/ に配置）
git clone <this-repo> ~/.claude/skills/pm-blueprint
# Claude Code を再起動
```

## 使い方

### 基本フロー

```
1. プロジェクトの RFP / 要件概要を用意
2. Claude Code で「pm-blueprint を使って [RFP概要] から計画書を作って」と依頼
3. custom/統合オーケストレーター.md が自動起動
4. 以下のステップが順次実行される（推奨フロー）:
   - Step 1: Layer 2 ディスカバリー
   - Step 2: Layer 2 前提抽出
   - Step 3: Layer 3 ADR 作成 + データ設計詳細
   - Step 4: Layer 4 ユースケース + SMART-NFR + EARS
   - Step 5: Layer 5 事前検死 + リスクレジスタ + 脅威モデリング
   - Step 6: Layer 7 法務・コンプライアンス対応
   - Step 7: Layer 8 LLM ガバナンス
   - Step 8: Layer 9 運用設計
   - Step 9: Layer 1 意思決定テンプレート
   - Step 10: Layer 6 WBS テンプレート + トレーサビリティ
5. 最終的に 12 セクション以上のプロジェクト計画書一式が生成される
```

詳細：[examples/サンプル適用例.md](examples/サンプル適用例.md)

## ディレクトリ構造

```
skill-pm-blueprint/
├── SKILL.md
├── README.md
├── INSTALL.md
├── licenses/
│   └── NOTICE.md
├── layer-1-executive/        # 経営層視点
├── layer-2-hypothesis/       # 仮説・戦略
├── layer-3-architecture/     # アーキテクチャ
├── layer-4-requirements/     # 要件定義
├── layer-5-risk/             # リスク（思考フレームワーク含む）
│   └── thinking-frameworks/  # ケプナートリゴー・可逆性分析 等
├── layer-6-execution/        # 実行 WBS
├── layer-7-legal-compliance/ # 法務・コンプラ
├── layer-8-llm-governance/   # LLM ガバナンス
├── layer-9-operations/       # 運用設計
├── custom/                   # 独自拡張
├── templates/                # ひな形（19+ 種類）
└── examples/                 # 架空プロジェクトでの適用例
```

## 注意事項

- このスキルは「**机上で青写真を描く**」ことが目的。実装フェーズのスクラム運用は別スキルで補完する想定
- 仮説は仮説であることを明示（未確定事項には `[仮定]` タグを付ける）
- テンプレートのプレースホルダ `[埋めてください]` は実プロジェクトの値で必ず置換する
- 矢羽パターンは本スキルのデフォルトだが、純粋ウォーターフォールや純粋アジャイルにも適用可能
- 日本の組織文化への配慮は `custom/日本型PMコンテキスト.md` を参照

## ライセンス

**Apache License 2.0**（[../LICENSE](../LICENSE)）

複数の OSS から着想・構造を借用しており、個別出典は `licenses/NOTICE.md` を参照。

## 作成・由来

- 2026-04-24 初版（6 レイヤー）
- 2026-04-30 改訂（Layer 7-9 追加 + agentic-quality-gate との連携強化）
- 対象: Claude Code (Agent SDK)

## 関連リンク

- 親プロジェクト：[agentic-quality-gate](../README.md)
- 解説記事：[Vibe Coding を Enterprise 品質まで引き上げる Agentic Quality Gate ── Karpathy の宿題に、プロ SIer の答案を](https://qiita.com/) *（公開準備中）*
