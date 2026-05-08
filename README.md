# Agentic Quality Gate

> **Vibe Coding を Enterprise 品質まで引き上げる、自己進化する品質ゲート群**

Andrej Karpathy 氏が 2026 年 4 月の Sequoia AI Ascent で再定義した **Agentic Engineering** ──「品質バーを下げずにエージェントで加速する工学規律」── を、**プロ SI ベンダーがエンタープライズ顧客に納品できる品質まで持続させる** ための、Claude Code 上の実装と PoC 実証。

🔗 **解説記事（Qiita）**: [Vibe Coding を Enterprise 品質まで引き上げる Agentic Quality Gate ── Karpathy の宿題に、プロ SIer の答案を](https://qiita.com/) *（公開準備中）*

---

## 何が含まれるか

本リポジトリには、Agentic Quality Gate を構成する 4 つの成果物が含まれます。**いずれも Apache-2.0 で OSS として公開**します。

| ディレクトリ | 内容 | 状態 |
|---|---|---|
| [`plugin/`](plugin/) | **Claude Code Plugin** ── 8 Subagent / 6 Skill / 6 Slash Command / 176 件のナレッジ + Excel ドライバ (`master.xlsx`) | ✅ **公開済み**（Claude Security の本実装は後続リリース） |
| [`skill-pm-blueprint/`](skill-pm-blueprint/) | **pm-blueprint Skill** ── プロジェクト計画書を 9 レイヤ（Executive / Hypothesis / Architecture / Requirements / Risk / Execution / Legal-Compliance / LLM-Governance / Operations）で自律生成するスキル | ✅ **公開済み** |
| [`docs/`](docs/) | **アーキテクチャ・QuickStart 等** ── 全体図、6 週間 MVP の進め方 | 🟡 概要のみ公開、詳細は順次追加 |

> Phase A の現時点では、**README とライセンスのみ**を公開しています。Phase B で各サブディレクトリの中身を順次充填します。

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

#### インストール

```bash
git clone https://github.com/kiwiiosaru-jp/agentic-quality-gate.git ~/.claude/plugins/agentic-quality-gate
# Claude Code を再起動
```

#### 主要なスラッシュコマンド

| コマンド | 用途 |
|---|---|
| `/aqg:evaluate /path/to/project` | **4 Phase パイプラインで成果物を品質評価**（Sensing → KB Refresh → Evaluation → Feedback） |
| `/aqg:sense` | **外部変化サーチ**（NVD / GHSA / IPA / JPCERT / OWASP / PIPC / EDPB / HackerNews / Bluesky / GitHub Trending / Postmortems / X / arXiv / クラウド料金 / LLM Provider Release の 14 ソース） |
| `/aqg:incident` | **インシデント記録**（FP / 誤検知 / 気づきを構造化して蓄積） |
| `/aqg:reflect` | **内省サイクル実行**（蓄積したインシデント・FP 率から候補ナレッジを再生成） |
| `/aqg:checklist --phase=p2 --severity=critical,high` | **人間レビュー用チェックリスト出力**（フェーズ × 重大度で絞込） |
| `/aqg:claude-security` | **Claude Security 連携（スケルトン）** ── 後続リリースで本実装 |

#### 進化メカニズム

- **Reactive**：評価実行 or `/aqg:sense` 起動時に外部変化を取込み → master.xlsx の `candidates` シートに候補追加 → 人間レビュー → 採用なら Checklist 昇格 → MD 再生成
- **Reflective**：`/aqg:reflect` で 4 モード分析（incidents 抽象化 / FP 率高エントリ検出 / Conditional 観点抽出 / 横断トレンド）→ 候補追加 → 人間レビュー

詳細：[`plugin/README.md`](plugin/README.md)

---

### 2. プロジェクト計画自動作成（`pm-blueprint` Skill）の使い方

**RFP / 要件概要から、プロジェクト計画書一式を 9 レイヤーで自律生成する Claude Code Skill** です。

#### インストール

```bash
git clone https://github.com/kiwiiosaru-jp/agentic-quality-gate.git
ln -s "$(pwd)/agentic-quality-gate/skill-pm-blueprint" ~/.claude/skills/pm-blueprint
# Claude Code を再起動
```

#### 基本フロー

1. プロジェクトの **RFP / 要件概要** を用意
2. Claude Code で「`pm-blueprint` を使って [RFP 概要] から計画書を作って」と依頼
3. `skill-pm-blueprint/custom/統合オーケストレーター.md` が自動起動
4. 以下のステップが順次実行される（推奨フロー）:
   - **Step 1**: Layer 2 ディスカバリー
   - **Step 2**: Layer 2 前提抽出
   - **Step 3**: Layer 3 ADR 作成 + データ設計詳細
   - **Step 4**: Layer 4 ユースケース + SMART-NFR + EARS
   - **Step 5**: Layer 5 事前検死 + リスクレジスタ + 脅威モデリング
   - **Step 6**: Layer 7 法務・コンプライアンス対応
   - **Step 7**: Layer 8 LLM ガバナンス
   - **Step 8**: Layer 9 運用設計
   - **Step 9**: Layer 1 意思決定テンプレート
   - **Step 10**: Layer 6 WBS テンプレート + トレーサビリティ
5. 最終的に **12 セクション以上のプロジェクト計画書一式** が生成されます

詳細：[`skill-pm-blueprint/README.md`](skill-pm-blueprint/README.md) ／ [`skill-pm-blueprint/examples/サンプル適用例.md`](skill-pm-blueprint/examples/サンプル適用例.md)

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

## 著者・関連リンク

- 著者：ちゅらデータ株式会社
- Qiita Organization: [ちゅらデータ](https://qiita.com/) *（リンク準備中）*
- 関連：[Andrej Karpathy — Sequoia Ascent 2026 summary](https://karpathy.bearblog.dev/sequoia-ascent-2026/)

---

## 貢献

Issue / PR は歓迎します。設計議論は Discussions にてお願いします。詳細な貢献ガイドラインは Phase B で整備予定。

---

> **Quality as Code × Living Spec × Agentic Evolution**
