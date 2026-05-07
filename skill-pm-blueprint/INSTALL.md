# pm-blueprint インストール手順

## 前提

- macOS / Linux / Windows いずれか
- Claude Code が導入済みであること
- `~/.claude/skills/` ディレクトリへの書き込み権限があること

## インストール

本スキルは既に `~/.claude/skills/pm-blueprint/` に配置されているため、追加作業は不要です。

### 確認

```bash
ls ~/.claude/skills/pm-blueprint/
```

以下のディレクトリが見えれば OK:

```
SKILL.md  README.md  INSTALL.md  licenses/
layer-1-executive/  layer-2-hypothesis/  layer-3-architecture/
layer-4-requirements/  layer-5-risk/  layer-6-execution/
custom/  templates/  examples/  test-results/
```

### Claude Code への反映

既に起動中の Claude Code セッションではスキルが認識されていない可能性があります。**Claude Code を再起動**してください。再起動後、新規セッションで以下のように問い合わせると認識状況が確認できます:

```
pm-blueprint スキルは利用可能か?
```

## 動作確認

インストール後、以下のコマンドで簡易動作確認ができます:

```
pm-blueprint を使って、以下のRFPから計画書を作成してください:
[架空のRFP]
- プロジェクト名: サンプル分析基盤
- 予算: 1億円
- 期間: 12ヶ月
- ドメイン: EC データ分析
```

Claude Code が `custom/統合オーケストレーター.md` を起点に7ステップを進めれば成功です。

## アップデート

本スキルを更新する場合:

1. `~/.claude/skills/pm-blueprint/` の該当ファイルを直接編集
2. Claude Code を再起動 (スキルメタデータ再読込のため)

## アンインストール

```bash
rm -rf ~/.claude/skills/pm-blueprint/
```

## トラブルシューティング

| 症状 | 対処 |
|------|------|
| スキルが認識されない | Claude Code を再起動する |
| `SKILL.md` 先頭の YAML frontmatter が読めない | ファイル先頭に `---` から始まるブロックがあるか確認 |
| サブスキル (layer-*) が個別に呼ばれない | オーケストレーターから呼ぶ想定。個別呼び出しは `layer-5-risk の事前検死を使って…` のように明示する |
| 出力が日本語にならない | プロンプトで「日本語で出力」を明示する |

## 関連ドキュメント

- `README.md` - 概要と使い方
- `SKILL.md` - スキルメタデータと全体像
- `licenses/NOTICE.md` - 参照OSSと出典

---

最終更新: 2026-04-24
