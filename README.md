# Agentic Quality Gate

> **Vibe Coding を Enterprise 品質まで引き上げる、自己進化する品質ゲート群**

Andrej Karpathy 氏が 2026 年 4 月の Sequoia AI Ascent で再定義した **Agentic Engineering** ──「品質バーを下げずにエージェントで加速する工学規律」── を、**プロ SI ベンダーがエンタープライズ顧客に納品できる品質まで持続させる** ための、Claude Code 上の実装と PoC 実証。

🔗 **解説記事（Qiita）**: [Vibe Coding を Enterprise 品質まで引き上げる Agentic Quality Gate ── Karpathy の宿題に、プロ SIer の答案を](https://qiita.com/) *（公開準備中）*

---

## 何が含まれるか

本リポジトリには、**2 つのメインコンポーネント** と補助ドキュメントが含まれます。**いずれも Apache-2.0 で OSS として公開**しています。

### 1. 品質ゲート ── [`plugin/`](plugin/)

**Claude Code Plugin（agentic-quality-gate）**。任意のプロジェクトに対して、品質ゲートが自律的に評価・サーチ・学習を行います。

- **8 Subagent**：gate-evaluator / signal-sensor / reflective-curator / feedback-collector / incident-recorder / project-explorer / report-writer / claude-security-adapter（スケルトン）
- **6 Skill**：checklist / evaluate-project / reflect / sense / incident / claude-security（スケルトン）
- **6 Slash Command**：`/aqg:checklist` `/aqg:evaluate` `/aqg:reflect` `/aqg:sense` `/aqg:incident` `/aqg:claude-security`
- **176 件のナレッジ**（[`master.xlsx`](plugin/knowledge/master.xlsx) + Markdown 自動派生）
- **状態**：✅ 公開済み（Claude Security 統合の本実装は後続リリース）

### 2. プロジェクト計画自動作成 ── [`skill-pm-blueprint/`](skill-pm-blueprint/)

**Claude Code Skill（pm-blueprint）**。RFP / 要件概要から、プロジェクト計画書一式を 9 レイヤーで自律生成します。

- **9 レイヤー**：Executive / Hypothesis / Architecture / Requirements / Risk / Execution / Legal-Compliance / LLM-Governance / Operations
- **テンプレート 19 種**（ADR / リスクレジスタ / WBS 雛形 / OpenAPI 仕様書 / SLO 定義書 等）
- **架空企業の適用例** ([`examples/サンプル適用例.md`](skill-pm-blueprint/examples/サンプル適用例.md))
- **状態**：✅ 公開済み

### 補助ドキュメント ── [`docs/`](docs/)

アーキテクチャ全体図、QuickStart、6 週間 MVP の進め方など。**状態**：🟡 概要のみ公開、詳細は順次追加。

---

## 設計思想（4 設計原理）

| 原理 | 一言 |
|---|---|
| 📜 **Living Spec** | ルール = コード。Markdown + frontmatter で Git 管理 |
| 👥 **Dual-Use** | 1 ナレッジ = AI 評価ルール + 人間チェックリストの単一ソース |
| 🔁 **Auto-Update** | 外部信号（CVE / 規制 / 事例）を 24h 以内にルール候補 PR へ |
| 🔐 **Human-in-Loop** | critical / 法務 / 認可 / PII は人間レビュー必須。完全自動化はしない |

詳細は Qiita 記事を参照ください。

---

## アーキテクチャ概観

```
外部世界 → ① Sensing → ② Normalize → ③ Knowledge Base → ④ Synthesis → ⑤ Evaluation → ⑥ Feedback
            (CVE/規制/事例)              (Git管理MD)        (PR候補)        (Hooks/Subagents)    (TP/FP/インシデント)
                                                                                      ↑
                                                                  Feedback Loop が ②③④ にも循環
```

7 + 1 フェーズゲート（P0 構想 → P6 運用 + Cross-cutting）に紐付けて、Hook / Schedule / Skill が連動。

---

## 使い方

このリポジトリには **2 つのメインコンポーネント** が入っています。それぞれ単独でも使えますし、組み合わせて連携運用するのが推奨です。

### 1. 品質ゲート（`agentic-quality-gate` Plugin）の使い方

**任意のプロジェクトに対して、品質ゲートが自律的に評価・サーチ・学習を行う Claude Code Plugin** です。

#### 一番シンプルな使い方

> **任意のフォルダにレビューしたい成果物一式（RFP・計画書・設計書・コード・ADR 等、何でも）を格納し、Claude Code に自然言語で「このフォルダの成果物をレビューして」と指示するだけ** ── あとは Plugin が次を全部やります：
>
> 1. **成果物の内容を自律的に把握**（フォルダ構造・命名・技術スタックを探索、ハードコードされたパス前提なし）
> 2. **該当するフェーズ（P0 構想 / P1 設計 / P2 実装 / … / Cross-cutting）を特定**
> 3. **そのフェーズで適用すべきチェック観点・評価指標を、ナレッジベース（176 件）から自動参照**
> 4. **各観点について Pass / Conditional / Fail / N/A を引用付きで判定**
> 5. **評価結果を Markdown 報告書として出力**（経営層 / レビュアー / 開発者の 3 視点で構造化）

#### インストール

```bash
git clone https://github.com/kiwiiosaru-jp/agentic-quality-gate.git ~/.claude/plugins/agentic-quality-gate
# Claude Code を再起動
```

#### 実行例

```bash
# 一番シンプル
/aqg:evaluate ~/projects/my-rfp-and-plan

# あるいは、自然言語で
「~/projects/my-rfp-and-plan にある成果物を agentic-quality-gate でレビューして」

# スコープを絞る（提案書・計画書レビュー）
/aqg:evaluate ~/projects/my-rfp-and-plan --scope=plan_review

# 重大度で絞る（critical と high のみ）
/aqg:evaluate ~/projects/my-app --scope=code_review --severity=critical,high
```

実行すると、`reports/{timestamp}_{プロジェクト名}.md` に評価結果が出力されます。

#### 他の主要なスラッシュコマンド

| コマンド | 用途 |
|---|---|
| `/aqg:sense` | **外部変化サーチ**（NVD / GHSA / IPA / JPCERT / OWASP / PIPC / EDPB / HackerNews / Bluesky / GitHub Trending / Postmortems / X / arXiv / クラウド料金 / LLM Provider Release の 14 ソース）。tech_stack 関連度を採点して候補ナレッジを生成 |
| `/aqg:incident` | **インシデント記録**（FP / 誤検知 / 気づきを構造化して蓄積） |
| `/aqg:reflect` | **内省サイクル実行**（蓄積したインシデント・FP 率から候補ナレッジを再生成） |
| `/aqg:checklist --phase=p2 --severity=critical,high` | **人間レビュー用チェックリスト出力**（フェーズ × 重大度で絞込） |
| `/aqg:claude-security` | **Claude Security 連携（スケルトン）** ── 後続リリースで本実装 |

#### 進化メカニズム

- **Reactive**：評価実行 or `/aqg:sense` 起動時に外部変化を取込み → `master.xlsx` の `candidates` シートに候補追加 → 人間レビュー → 採用なら Checklist 昇格 → MD 再生成
- **Reflective**：`/aqg:reflect` で 4 モード分析（incidents 抽象化 / FP 率高エントリ検出 / Conditional 観点抽出 / 横断トレンド）→ 候補追加 → 人間レビュー

詳細：[`plugin/README.md`](plugin/README.md)

---

### 2. プロジェクト計画自動作成（`pm-blueprint` Skill）の使い方

**RFP / 要件概要から、プロジェクト計画書一式を 9 レイヤーで自律生成する Claude Code Skill** です。

#### 一番シンプルな使い方

> **任意のフォルダに RFP（提案依頼書）や要件概要のテキストを置き、Claude Code に自然言語で「この RFP からプロジェクト計画書一式を作って」と指示するだけ** ── あとは Skill が次を全部やります：
>
> 1. **RFP / 要件概要を読み解いて、不足情報を仮説で補う**（前提抽出 / ディスカバリー、未確定事項には `[仮定]` タグを付与）
> 2. **アーキテクチャ判断（ADR）・データ設計・API 設計を起こす**
> 3. **機能要件（ユースケース）・非機能要件（SMART-NFR・EARS 形式）を整理**
> 4. **リスク洗い出しと Kill 基準を設定**（事前検死・リスクレジスタ・脅威モデリング）
> 5. **法務・コンプライアンス対応 / LLM ガバナンス / 運用設計まで網羅**
> 6. **投資判断（Go/No-Go）と矢羽パターンの WBS を生成**
> 7. **最終的に 12 セクション以上のプロジェクト計画書一式を Markdown で出力**（経営判断資料・アーキ図・要件・リスク・WBS・トレーサビリティマトリクスまで）

#### インストール

```bash
git clone https://github.com/kiwiiosaru-jp/agentic-quality-gate.git
ln -s "$(pwd)/agentic-quality-gate/skill-pm-blueprint" ~/.claude/skills/pm-blueprint
# Claude Code を再起動
```

#### 実行例

```bash
# 一番シンプル
「pm-blueprint を使って ~/projects/my-rfp/RFP.md から計画書を作って」

# 既存フォルダ全体を読ませる
「pm-blueprint を使って ~/projects/my-project/ にある RFP と要件メモから計画書一式を作って」

# 特定レイヤーだけ生成したい場合
「pm-blueprint の Layer 5（リスク）だけで、リスクレジスタを作って」
```

実行すると、`skill-pm-blueprint/custom/統合オーケストレーター.md` が自動起動し、以下の **10 ステップ** が順次実行されます。

#### 内部の詳細フロー（10 ステップ）

| Step | レイヤー | 中身 |
|---:|---|---|
| 1 | Layer 2 | ディスカバリー（JTBD・ステークホルダー・OST 優先解決策） |
| 2 | Layer 2 | 前提抽出（4 軸前提分析 V/U/Vi/F、不確実性マトリクス、検証計画） |
| 3 | Layer 3 | ADR 作成 + データ設計詳細（C4 図、コンテキストマップ） |
| 4 | Layer 4 | ユースケース + SMART-NFR + EARS（FURPS+ ランディングゾーン） |
| 5 | Layer 5 | 事前検死 + リスクレジスタ + 脅威モデリング（STRIDE / LINDDUN） |
| 6 | Layer 7 | 法務・コンプライアンス対応（個情法対応マトリクス・業法該当性判定書 等） |
| 7 | Layer 8 | LLM ガバナンス（PII 境界 / 出力検証 / エージェント権限境界書 等） |
| 8 | Layer 9 | 運用設計（AI データ境界 / シャドー AI 禁止 / ランブック規約） |
| 9 | Layer 1 | 意思決定テンプレート（Go/No-Go 判断資料、シナリオ別期待値） |
| 10 | Layer 6 | WBS テンプレート + トレーサビリティマトリクス（矢羽 L2/L3/L4） |

詳細：[`skill-pm-blueprint/README.md`](skill-pm-blueprint/README.md) ／ 架空企業での適用例：[`skill-pm-blueprint/examples/サンプル適用例.md`](skill-pm-blueprint/examples/サンプル適用例.md)

---

### 1 + 2 の連携利用（推奨）

提案書作成や PoC 立ち上げの場面では、**「pm-blueprint で生成 → agentic-quality-gate で評価 → 改善ループ」** が最大効果を発揮します。

```
RFP / 要件概要
    ↓
pm-blueprint Skill が計画書一式を自律生成（12+ セクション）
    ↓
agentic-quality-gate Plugin（/aqg:evaluate）で 176 観点の評価
    ↓
致命的問題があれば → 改善指示を pm-blueprint に戻す → 再生成
    ↓
品質基準達成まで自動ループ（人間は方針承認のみ）
```

このループの実例（致命的問題 13 → 0 件 を一晩で達成、など）は、本リポジトリと連動する Qiita 記事で解説しています：

🔗 *Qiita 記事 URL は公開準備中（公開後にここへ追記）*

---

## 6 週間 MVP（推奨ロードマップ）

```
Week 1   Step A : 種ナレッジ整備（Critical 38 件 MD 作成、INDEX 自動生成）
Week 2-3 Step B : 人間消費パス（/checklist skill 実装、PR レビュー実用）
Week 3-4 Step C : Hook 強制（PreToolUse / PostToolUse / Stop / SessionStart）
Week 5-6 Step D : Curator 自動化（ingest mode + Shadow mode 1 週間運用）
```

---

## ライセンス

**Apache License 2.0** — 商用利用・改変・再配布可、特許クレーム保護を含む。詳細は [LICENSE](LICENSE) 参照。

---

## 関連リンク

- 解説記事（Qiita）: *公開準備中（公開後にここへ追記）*

---

## 貢献

Issue / PR は歓迎します。設計議論は Discussions にてお願いします。詳細な貢献ガイドラインは Phase B で整備予定。

---

> **Quality as Code × Living Spec × Agentic Evolution**
