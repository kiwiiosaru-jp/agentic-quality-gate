# Agentic Quality Gate (AQG)

外部環境変化（CVE / 規制 / インシデント / コスト構造）と自社経験にニアリアルタイムで追随する **生きた品質ゲート** を、Claude Code 駆動で実現する仕組み。

> 「持っているのに守れない」を「自動で動き続けるから守られる」へ。

## 設計原則

| 原則 | 実装 |
|---|---|
| **AI ファースト** | AIエージェントが自律的にプロジェクトを解析・判定。Humanは結果のレビューに専念 |
| **自律探索** | プロジェクトのルートフォルダだけ指定すれば、AIが Glob/Grep/Read で関連資料を探す |
| **スキル定義ベース** | ルールハードコードではなく、Subagent + Skill で柔軟判断 |
| **生きたナレッジ** | Excel マスター + Markdown 派生で Git管理、進化可能 |
| **Dual-Use** | 同じナレッジを AI 評価基準 と 人間レビュー観点 の双方で使う |
| **Reactive + Reflective** | 外部信号取込（Reactive）と 自社経験フィードバック（Reflective）の両輪 |

## アーキテクチャ概要（6層）

```
        外部信号                    自社経験
   ┌─────────────────┐          ┌──────────────┐
   │ NVD / OWASP     │          │ Incidents    │
   │ PIPC / IPA      │          │ Post-mortems │
   │ HackerNews      │          │ FP/FN logs   │
   │ Bluesky         │          │ Reviews      │
   │ X (検索)        │          │              │
   └────────┬────────┘          └──────┬───────┘
            │                          │
            ▼                          ▼
   ┌─────────────────────────────────────────────┐
   │ Layer 1+2: Sensing & Normalize              │
   │   signal-sensor agent (Reactive)            │
   └────────────────┬────────────────────────────┘
                    │ candidates (status=candidate)
                    ▼
   ┌─────────────────────────────────────────────┐
   │ Layer 3: Knowledge Base                     │
   │   master.xlsx (Excel = マスター)             │
   │   knowledge/*.md (Markdown = 派生)           │
   └────┬────────────────────────────────────────┘
        │
        ├──→ ┌─────────────────────────────────┐
        │    │ Layer 5: Evaluation Engine      │
        │    │  project-explorer →             │
        │    │  gate-evaluator →               │
        │    │  report-writer                  │
        │    └────────────────┬────────────────┘
        │                     │ verdicts
        │                     ▼
        │    ┌─────────────────────────────────┐
        │    │ Layer 6: Feedback Loop          │
        │    │  feedback-collector             │
        │    │  + incident-recorder            │
        │    │  + reflective-curator           │
        │    └────────────────┬────────────────┘
        │                     │
        └─────────────────────┘  ← incidents → candidates へ循環
```

## ディレクトリ構成

```
agentic-quality-gate/
├── .claude-plugin/plugin.json    ← Claude Code Plugin マニフェスト
├── agents/                        ← 5 Subagent
│   ├── project-explorer.md       ← プロジェクト構造を自律解析
│   ├── gate-evaluator.md         ← ナレッジと突合判定
│   ├── report-writer.md          ← 品質報告書を生成
│   ├── signal-sensor.md          ← Layer 1+2 (Reactive)
│   ├── feedback-collector.md     ← Layer 6 (effectiveness 集計)
│   ├── incident-recorder.md     ← 自社経験記録
│   └── reflective-curator.md    ← Layer 6 (Reflective 抽象化)
├── skills/                        ← 5 User-invocable Skill
│   ├── evaluate-project/SKILL.md ← 4 Phase 評価パイプライン
│   ├── checklist/SKILL.md
│   ├── sense/SKILL.md            ← 外部変化サーチのみ
│   ├── incident/SKILL.md         ← 経験記録
│   └── reflect/SKILL.md          ← 内省サイクル
├── commands/                      ← 5 Slash Command
│   ├── evaluate.md, checklist.md, sense.md
│   ├── incident.md, reflect.md
├── knowledge/
│   ├── master.xlsx               ← マスター（編集元）
│   │   ├── Checklist             (176件、active のみMD化)
│   │   ├── Summary, README
│   │   ├── candidates            (signal-sensor / reflective-curator が追加)
│   │   ├── senses                (外部信号取込履歴)
│   │   ├── incidents             (自社経験)
│   │   ├── effectiveness         (TP/FP/skipped 集計)
│   │   └── reflections           (内省サイクル実行履歴)
│   ├── INDEX.md, schema.yaml
│   ├── phases/{P0..P6}/*.md     ← excel_to_md.py で再生成
│   └── cross-cutting/{...}/*.md
├── scripts/
│   ├── add_meta_sheets.py
│   ├── add_incidents_sheet.py
│   └── excel_to_md.py
├── examples/
└── reports/                      ← 生成された各種レポート
```

## 使い方

### 1. プラグインを有効化

```bash
# このディレクトリ（plugin/）を ~/.claude/plugins/agentic-quality-gate/ にクローン or シンボリックリンク
git clone <THIS_REPO_URL> ~/.claude/plugins/agentic-quality-gate
# または
ln -s "$(pwd)" ~/.claude/plugins/agentic-quality-gate
# Claude Code を再起動
```

### 2. プロジェクトを評価（4 Phase パイプライン）

```bash
/aqg:evaluate /path/to/your/project
```

→ Phase 0 (Sensing) → Phase 1 (KB Refresh) → Phase 2 (Evaluation) → Phase 3 (Feedback) を一気通貫実行

### 3. 外部変化サーチのみ（評価なし）

```bash
/aqg:sense                          # 全信号源
/aqg:sense --tech=python,llm,rag    # 自社stack絞り込み
```

### 4. インシデント記録

```bash
/aqg:incident                                                # 対話モード
/aqg:incident type=incident project="VoC PoC" summary="..."  # 一括指定
```

### 5. 内省サイクル実行

```bash
/aqg:reflect                              # 全モード
/aqg:reflect --mode=incidents-only        # 経験のみ
/aqg:reflect --mode=effectiveness-only    # FP分析のみ
```

### 6. チェックリスト出力（人間レビュー用）

```bash
/aqg:checklist --phase=p2 --severity=critical,high
```

## 仕組みの中核

### Reactive 進化（外部起点）

```
評価実行 or /aqg:sense
    ↓
signal-sensor が WebFetch / WebSearch で14ソース取得：
  - 公式: NVD, GHSA, IPA, JPCERT, OWASP, PIPC, EDPB
  - 業界: HackerNews, Bluesky, GitHub Trending, Postmortems, X検索, arXiv
  - コスト: クラウド料金、LLM Provider更新
    ↓
自社stack関連度を採点 → master.xlsx の candidates シートに追加
    ↓
人間レビュー → 採用なら Checklist 昇格 → MD再生成
```

### Reflective 進化（内部起点）

```
インシデント発生 → /aqg:incident で記録
評価のたびに    → effectiveness シート自動更新（feedback-collector）
    ↓
/aqg:reflect 実行（任意のタイミング、月次推奨）
    ↓
reflective-curator が4モードで分析：
  - Mode 1: incidents 抽象化（同種パターン抽出）
  - Mode 2: FP率高エントリの曖昧さ検出
  - Mode 3: 過去評価の Conditional 観点抽出
  - Mode 4: 横断トレンド（incidents × senses 相関）
    ↓
candidates シートに追加 → 人間レビュー → 採用なら Checklist 昇格
```

## クロール対象（14ソース、Claude Codeネイティブで完結）

| カテゴリ | ソース | アクセス方法 |
|---|---|---|
| **A. 脆弱性・脅威** | NVD CVE, GitHub Security Advisories, IPA, JPCERT, OWASP Top 10 (Web/LLM) | WebFetch |
| **B. 規制・法務** | PIPC, EDPB (GDPR), 業法 | WebFetch / WebSearch |
| **C. 業界・現場感（SNS含む）** | HackerNews (Algolia API), Bluesky (Public API), GitHub Trending, Postmortems repo, X (WebSearch site:x.com), arXiv | WebFetch / WebSearch |
| **D. コスト・運用** | クラウド料金変更, LLM Provider Release Notes | WebFetch / WebSearch |

外部MCPは不要。Claude Code ネイティブの `WebFetch` と `WebSearch` のみで動作。

## 進化メカニズム

| モード | トリガ | 起動 |
|---|---|---|
| **Reactive** | 外部信号 | 評価実行時 / `/aqg:sense` 手動 |
| **Reflective** | 自社経験 | `/aqg:reflect` 手動 |
| ~~Proactive~~ | スケジュール | （ペンディング、当面は手動 reflect で代替） |

## ライセンス

**Apache License 2.0**（[../LICENSE](../LICENSE)）── 商用利用・改変・再配布可、特許クレーム保護を含みます。
