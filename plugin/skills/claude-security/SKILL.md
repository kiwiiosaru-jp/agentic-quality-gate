---
name: claude-security
description: |
  Import Claude Security (Anthropic 2026-04-30 Public Beta) findings into agentic-quality-gate. Bridge between Anthropic's dedicated code-security AI and the cross-cutting quality gate.
  Examples: "Claude Securityの結果取込んで", "/aqg:claude-security import findings.csv", "コードセキュリティの結果を統合".
---

# Skill: claude-security

Claude Security のスキャン結果を本仕組みに取り込むためのブリッジ。

## 起動条件

- 「Claude Security の結果取込」
- 「セキュリティスキャン結果統合」
- 「/aqg:claude-security」
- Webhook が `reports/claude_security_inbox/` に新ファイルを置いた時

## 利用前提（Admin 設定が必要）

このSkillは Claude Security が組織で有効化されていないと意味のあるデータを扱えません。状態確認：

| 確認項目 | 確認URL/方法 |
|---|---|
| Org が Claude Enterprise | claude.ai のヘッダにorg名が表示されていればOK |
| Claude Code on the Web 有効 | claude.ai/code が開ける |
| Claude Security 有効化 | claude.ai/security が開ける |
| 自分のシートが Premium | Org admin に確認 |
| GitHub App 連携 | claude.ai/security でリポ選択画面が出る |

未設定なら、本Skillはサンプルデータでの動作確認モードに切替（後述）。

## 入力（必須）

以下のいずれか：

1. **`csv_path`**: Claude Security UI からエクスポートしたCSV
2. **`markdown_path`**: Claude Security UI からエクスポートしたMarkdown
3. **`webhook_payload_dir`**: webhook が蓄積したJSONディレクトリ（デフォルト: `reports/claude_security_inbox/`）
4. **`sample`**: `true` なら `examples/claude_security_sample.json` を使う（動作確認用）

## オプション

- **`auto_merge`**: `true` で gate-evaluator の評価結果と自動統合（デフォルト false、まずreviewする）
- **`update_effectiveness`**: `true` で master.xlsx の effectiveness シートにも反映（デフォルト true）

## 実行手順

`claude-security-adapter` agent を Task で起動：

```
入力:
  source_type: csv / markdown / webhook / sample
  source_path: <ファイルパス>
  master_xlsx: $CLAUDE_PLUGIN_ROOT/knowledge/master.xlsx
  auto_merge: <true/false>
```

## 出力

```
✅ Claude Security findings 取込完了

📊 取込
- HIGH: N件 → severity=critical, verdict=Fail
- MEDIUM: M件 → severity=high, verdict=Fail
- LOW: K件 → severity=medium, verdict=Conditional
- Dismissed: J件 (除外)

📁 出力
- /tmp/aqg_eval_claude_security.json
- master.xlsx の claude_security_findings シート: +N行
- master.xlsx の effectiveness シート: 自動反映 (auto_merge=true 時)

🎯 マッピング結果:
- SEC-INJECT-SQL-001: N件
- SEC-IDOR-001: M件
- ...

次のアクション:
→ 報告書化したい: /aqg:evaluate --merge-claude-security
→ effectiveness 反映: 自動済 (auto_merge=true)
→ 修正パッチを review: claude.ai/security の元findings を開く
```

## サンプルモード（動作確認）

Claude Security 未有効化の段階でも、本仕組み側は完全に準備可能です。

```bash
/aqg:claude-security --sample
```

これは `$CLAUDE_PLUGIN_ROOT/examples/claude_security_sample.json` を読み込み、本Skillの全パイプラインを通します。実 Claude Security が降りてくる前のテストに使えます。

## 設計思想

### Claude Security と本仕組みの役割分担

| 領域 | 担当 |
|---|---|
| コードベース脆弱性検出 (8カテゴリ) | **Claude Security** (Opus 4.7 Mythos) |
| 修正パッチ生成 | **Claude Security** |
| ナレッジ判定 (法務/コスト/設計/運用) | **agentic-quality-gate** |
| 横断的な品質統制 | **agentic-quality-gate** |
| 報告書統合 | **agentic-quality-gate** (本Skill経由で取込) |

### なぜ別プロダクトを統合するか

Anthropic が「Defender 専用ツール」として Claude Security を独立提供したのは、深い脆弱性チェーン解析には専用モデル（Mythos/Opus 4.7）が必要だから。本仕組みの gate-evaluator では同等精度は出ない。

→ 「コード脆弱性は Claude Security に委譲、本仕組みは横断品質のオーケストレーター」 という設計がベスト。

## 関連スキル

- `/aqg:evaluate` — Claude Security 結果も合流したフル評価
- `/aqg:checklist` — 人間レビュー観点（Claude Security findings も含む）
