---
name: checklist
description: |
  Generate a human review checklist (Markdown) filtered by phase, tech stack, and severity. Use for design review meetings, PR reviews, or pre-release audits.
  Examples: "P2のチェックリストを critical/high で出して", "LLM 関連のレビュー観点を一覧で".
---

# Skill: checklist

176件のナレッジから、人間レビューに使えるチェックリストを Markdown で抽出する。設計レビュー会議や PR レビューで配布する用途。

## 起動条件

- 「チェックリストを出して」
- 「設計レビューに使う観点を一覧で」
- 「P2 critical の項目だけ表示」
- 「/aqg:checklist」

## 入力

### 必須

なし（フィルタなしの場合は全件出力）

### オプション

- **`phase`**: フェーズ（複数可）。例: `["P2", "P3"]`, `"P2"`, `"all"`（デフォルト）
- **`tech_stack`**: 適用条件マッチ用。例: `["web", "llm", "rag"]`
- **`severity`**: `["critical", "high"]` がデフォルト（`"all"` で全件）
- **`output_format`**: `"markdown"`（デフォルト）/ `"github_issues"` / `"pdf"`
- **`output_path`**: 保存先（指定なければ標準出力）

## 実行手順

### Step 1: ナレッジ取得

```
1. knowledge/INDEX.md を Read
2. phases/{P}/*.md と cross-cutting/*/*.md を Glob
3. 各 .md の frontmatter を parse
```

### Step 2: フィルタ

- `phase`: frontmatter の `phase` フィールドと一致
- `severity`: frontmatter の `severity` がリストに含まれる
- `tech_stack`: frontmatter の `applies_when` を LLM文脈推論で判定
  - 例: `applies_when: "RAG/エージェント"` × tech_stack=["llm","rag"] → 適用
  - 例: `applies_when: "全プロジェクト"` × any tech → 必ず適用

### Step 3: Markdown出力

以下のフォーマットで出力：

```markdown
# 品質ゲート チェックリスト

**フィルタ**: Phase={...}, Tech={...}, Severity={...}
**生成日**: 2026-04-30
**適用エントリ数**: N 件

---

## 🔴 Critical（必須・Fail = リリース停止）

### [SEC-IDOR-001] IDOR：URLにIDがあるAPIの所有者照合

**観点**: URLやペイロードにリソースIDを含むAPIで、リクエスト元ユーザーがそのリソースの所有者・権限保有者であることを毎回サーバ側で検証しているか

**人間レビューチェック**:
- [ ] 全リソース系エンドポイントで認可検証コードの存在確認
- [ ] 「ログインしている」だけで通している箇所がないか
- [ ] 他ユーザIDで叩くテストケースの存在
- [ ] AI生成エンドポイントの目視レビュー実施

**OK基準**: 全リソース系エンドポイントにオーナーシップ検証コードが存在、かつ未認可アクセス試行のテストケースが全Pass

**📚 ナレッジ**: knowledge/phases/P2/SEC-IDOR-001.md

---

### [LEGAL-PII-001] 個人情報の定義・取扱目的・第三者提供整理
...

## 🟠 High（必須・Fail = リリース停止可能、例外承認可）
...

## 🟡 Medium（推奨）
...

## ⚠️ 棚卸し期限切れ（参考程度）
...
```

### Step 4: GitHub Issue 化（オプション）

`output_format: "github_issues"` の場合、各エントリを Issue として `gh issue create` で起票（dry-runオプションあり）。

## 出力例の特徴

- **チェック項目をそのまま使える**: コピペで PR レビューや会議に使える
- **重要度順**: Critical → High → Medium の順
- **OK基準も併記**: 判定の客観性のため
- **ナレッジへのリンク**: 詳細は元ファイルを参照

## 実用シナリオ

| シーン | フィルタ |
|---|---|
| 設計レビュー会議 | `phase=P0,P1` `severity=critical,high` |
| 実装レビュー（PR） | `phase=P2` `tech_stack=<該当技術>` |
| 本番リリース前点検 | `phase=P4,P5` `severity=critical,high` |
| LLM特化プロジェクト | `phase=cross-cutting` `tech_stack=llm,rag` |
| 法務観点の棚卸し | `phase=cross-cutting/compliance` |

---

## 関連スキル

- `/aqg:evaluate` — AIで実プロジェクトを評価（このチェックリスト基準）
- `/aqg:report` — 評価結果を報告書化
