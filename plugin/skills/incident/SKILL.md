---
name: incident
description: |
  Record an incident, false-positive, near-miss, or observation into the master.xlsx incidents sheet for reflective learning.
  Examples: "本番障害が起きた、記録して", "/aqg:incident", "AIの誤検知だった", "観点漏れに気づいた".
---

# Skill: incident

自社内の **品質関連の経験** を記録する。後で `reflective-curator` がパターン抽出・抽象化してナレッジ候補を生成する。

## 起動条件

- 「本番障害を記録」「インシデント記録」
- 「AIの誤検知だった」「false-positiveを記録」
- 「ニアミスがあった」「観点漏れに気づいた」
- 「/aqg:incident」

## 入力（対話で収集）

ユーザーから以下を順次収集（一度に全部聞かず、段階的に）：

### 必須

1. **incident_type**: 種別
   - `incident`: 実害があった事故
   - `false-positive`: AI評価が誤検知だった
   - `near-miss`: 障害寸前で気づいた
   - `observation`: 気づき・観点漏れ

2. **summary**: 1行要約（例: "Streamlit認証なしで Text2SQL が任意SQL実行可能"）

3. **project_name**: 対象プロジェクト名

### 任意（後でAIが補完可）

4. **severity**: critical / high / medium / low（不在なら medium）
5. **what_happened**: 何が起きたか（自由記述）
6. **root_cause**: 根本原因（不在ならAIが推定）
7. **related_evidence**: ファイル/PR/Issue/ログURL
8. **related_knowledge_ids**: 関連既存ID（不在ならAIが INDEX.md から推定）
9. **recorded_by**: 記録者（不在なら "anonymous"）

## 実行手順

### Step 1: 対話で情報収集

ユーザーが「本番障害があった」と言ったら：
1. まず種別を聞く（incident? false-positive? observation?）
2. 1行要約を聞く
3. プロジェクト名を聞く
4. 任意項目を「もし分かれば」と聞く（無理に聞き出さない）

すでに情報が揃っている場合（例: ユーザーが詳細を一気に書いた場合）、追加質問せずに次へ。

### Step 2: incident-recorder Subagent を起動

Task ツールで `incident-recorder` を呼出：

```
入力:
  incident_type: <収集>
  project_name: <収集>
  severity: <収集または medium>
  summary: <収集>
  what_happened: <収集または空>
  root_cause: <収集または空>
  related_evidence: <収集または空>
  related_knowledge_ids: <収集または空>
  recorded_by: <収集または "anonymous">
  master_xlsx: $CLAUDE_PLUGIN_ROOT/knowledge/master.xlsx
```

### Step 3: ユーザーへの確認・完了報告

```
✅ Incident 記録完了

ID: INC-20260430-001
Type: incident
Project: ABC社 顧客接点 PoC
Severity: critical

Summary: Streamlit認証なしで Text2SQL が任意SQL実行可能

Auto-detected related: SEC-IDOR-001, SEC-AUTHZ-FUNC-001, LLM-OUTPUT-TRUST-001

次のアクション:
→ 蓄積したら /aqg:reflect を実行して内省サイクルへ
→ 関連 incidents が複数集まれば、自動で抽象化・ナレッジ更新候補化されます
```

## ユースケース

| シーン | 例 |
|---|---|
| 本番障害発生時 | ポストモーテム作成と並行して記録 |
| AIレビューで誤検知 | 「これはFalse positive」と思ったら即記録 |
| レビュー会議で観点漏れ発見 | observation として記録 |
| ニアミス | デプロイ直前で気づいた事象を記録 |

## 関連スキル

- `/aqg:reflect` — 蓄積した incidents から候補ナレッジを生成
- `/aqg:evaluate` — 評価時に Conditional 多発エントリを自動 observation 化
