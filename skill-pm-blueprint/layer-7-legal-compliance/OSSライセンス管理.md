---
name: OSSライセンス管理
description: プロジェクトで使う OSS のライセンスを SBOM (Software Bill of Materials) で管理し、GPL/AGPL 等のコピーレフト系汚染を予防する。LEGAL-LICENSE-001 (Major Fail) 対応スキル。
---

# OSSライセンス管理

## 対応する品質評価ID

- **LEGAL-LICENSE-001 (Major)** - OSS ライセンスのSBOM管理と商用可否判定が未整備
- 関連: LEGAL-AIGEN-001 (学習データのライセンス)

## 概要

プロジェクトで利用する OSS (Open Source Software) のライセンスを**SBOM (Software Bill of Materials)** として管理し、商用利用可能性、ライセンス汚染リスクを評価する。

このスキルの目的:

1. **ライセンス可視化** - 全 OSS 部品のライセンス一覧を生成
2. **商用利用可否の判定** - GPL/AGPL 等のコピーレフト系を識別
3. **汚染リスクの予防** - 自社コードへの不本意な GPL 適用を回避
4. **ライセンス義務の整理** - 表示義務、ソース公開義務、特許条項
5. **法的リスクの評価** - ベンダーフリーの維持

## いつ使うか

- 矢羽②PoC+システム要件 (Week 7-12) - 技術選定確定時
- 新規 OSS ライブラリ追加時 (CI/CD で自動チェック推奨)
- 製品リリース前の最終チェック
- M&A デューデリジェンス時

## ライセンス分類

### コピーレフト (強)

派生物にも同一ライセンスを伝播させる。**自社コード公開義務**が発生する可能性。

| ライセンス | 商用可否     | 自社コード公開義務 | 備考                                       |
| ---------- | ------------ | ------------------ | ------------------------------------------ |
| GPL v2/v3  | OK (条件付き)| あり (リンク含)    | サーバー側のみで使う場合は不要 (例外あり) |
| AGPL v3    | OK (条件付き)| あり (SaaS でも)   | SaaS でも公開義務あり (最も厳しい)        |
| LGPL       | OK           | 動的リンクなら不要 | 静的リンクでは公開義務                    |

### コピーレフト (弱)

改変したファイルのみ公開義務。

| ライセンス | 商用可否 | 公開義務            | 備考                                |
| ---------- | -------- | ------------------- | ----------------------------------- |
| MPL 2.0    | OK       | 改変ファイルのみ    | Mozilla Public License              |
| EPL 2.0    | OK       | 改変ファイルのみ    | Eclipse Public License              |
| LGPL       | OK       | リンク方式に依存    | 動的リンクなら制限緩い              |

### 寛容型 (Permissive)

商用利用に最も柔軟。

| ライセンス     | 商用可否 | 表示義務          | 備考                                        |
| -------------- | -------- | ----------------- | ------------------------------------------- |
| MIT            | OK       | 著作権表示のみ    | 最も柔軟                                    |
| Apache 2.0     | OK       | 著作権 + NOTICE   | 特許条項あり (使う側にメリット)            |
| BSD 2/3-Clause | OK       | 著作権表示のみ    | 広告禁止条項 (旧 4-Clause)                 |
| ISC            | OK       | 著作権表示のみ    | OpenBSD で多用                             |
| Unlicense      | OK       | なし              | パブリックドメイン化                        |

### 商用制限・特殊

| ライセンス       | 商用可否            | 備考                                                       |
| ---------------- | ------------------- | ---------------------------------------------------------- |
| SSPL             | NG (要確認)         | MongoDB が採用、SaaS で要厳格な確認                       |
| BSL              | 制限あり            | CockroachDB 等、X 年経過後に Apache 2.0 化                |
| Elastic License  | 制限あり            | Elasticsearch 7.11+ は商用 SaaS 制限                      |
| Commons Clause   | NG (商用配布禁止)   | -                                                          |
| 独自ライセンス   | 個別判断            | 各製品の個別契約条項を確認                                |

## 手順

### ステップ 1: SBOM 自動生成

依存関係から SBOM を自動生成。CycloneDX または SPDX 形式を推奨。

#### 言語別の生成ツール

| 言語     | ツール例                                  |
| -------- | ----------------------------------------- |
| Python   | `pip-licenses`, `cyclonedx-python`       |
| Node.js  | `license-checker`, `cyclonedx-bom`       |
| Java     | `cyclonedx-maven-plugin`, `license-maven`|
| Go       | `go-licenses`, `cyclonedx-gomod`         |
| Rust     | `cargo-license`, `cargo-cyclonedx`       |
| 全言語   | `syft`, `trivy` (コンテナ含む)           |

### ステップ 2: ライセンス分類

生成された SBOM を以下の 3 区分に分類:

| 区分     | アクション                              |
| -------- | --------------------------------------- |
| OK       | そのまま利用可能 (記録のみ)             |
| 条件付き | 表示義務、配布制限等の確認 → 対応文書化 |
| NG       | 即時除去 + 代替 OSS 探索                |

### ステップ 3: ライセンス義務の整理

「条件付き OK」のライセンスについて、必要対応を整理:

- 著作権表示のクレジット集約 (例: `THIRD-PARTY-NOTICES.md`)
- ソース公開義務がある場合の公開先 (GitHub 等)
- 特許条項の確認 (特に Apache 2.0)
- 改変履歴の記録義務

### ステップ 4: 自動化された継続チェック

CI/CD パイプラインに OSS ライセンスチェックを組込む:

```yaml
# 例: GitHub Actions
- name: License Check
  run: |
    syft scan dir:. -o cyclonedx-json > sbom.json
    grype sbom:sbom.json --fail-on high
    license-checker --production --excludePackages "MyProject" --failOn "GPL;AGPL;SSPL"
```

### ステップ 5: 法務承認

「条件付き OK」「個別判断」のライセンスは、法務承認を経る:

```
プロジェクト責任者 起票
   ↓
情シス (技術評価)
   ↓
法務部 (ライセンス文面確認)
   ↓
SBOM 確定
```

## 例示

### データ分析基盤 (DWH/Lakehouse) のケース

| OSS                | バージョン | ライセンス    | 区分     | 備考                                   |
| ------------------ | ---------- | ------------- | -------- | -------------------------------------- |
| dbt-core           | 1.7.x      | Apache 2.0    | OK       | NOTICE ファイル付与                    |
| dbt-snowflake      | 1.7.x      | Apache 2.0    | OK       | -                                      |
| great-expectations | 0.18.x     | Apache 2.0    | OK       | -                                      |
| Apache Airflow     | 2.8.x      | Apache 2.0    | OK       | -                                      |
| Pandas             | 2.x        | BSD 3-Clause  | OK       | -                                      |
| MongoDB Driver     | 4.x        | Apache 2.0    | OK       | (MongoDB 本体は SSPL なので別途確認)  |
| Streamlit          | 1.x        | Apache 2.0    | OK       | -                                      |
| ChromaDB           | 0.4.x      | Apache 2.0    | OK       | ベクター DB                            |
| MinIO              | 2024.x     | AGPL v3       | NG       | → 代替: Azure Blob Storage に変更      |

MinIO が AGPL v3 のため、代替検討が必要。AGPL は SaaS でも自社コード公開義務が発生するため、商用 SaaS では原則使用しない。

### コールセンター (VoC 分析) のケース

| OSS                  | バージョン | ライセンス    | 区分     | 備考                                            |
| -------------------- | ---------- | ------------- | -------- | ----------------------------------------------- |
| Whisper (OpenAI)     | -          | MIT           | OK       | ASR 音声認識用                                  |
| Whisper.cpp          | -          | MIT           | OK       | 軽量実行用                                      |
| FasterWhisper        | -          | MIT           | OK       | 推論高速化                                      |
| Sentence-Transformers| 2.x        | Apache 2.0    | OK       | -                                               |
| sklearn              | 1.x        | BSD 3-Clause  | OK       | -                                               |
| TensorFlow           | 2.x        | Apache 2.0    | OK       | -                                               |
| PyTorch              | 2.x        | BSD 3-Clause  | OK       | -                                               |
| spaCy                | 3.x        | MIT           | OK       | -                                               |
| MeCab                | -          | BSD/LGPL/GPL  | 条件付き | NLP辞書もライセンス確認 (mecab-ipadic は GPL)  |

MeCab 自体は OK だが、**辞書の互換性**を要確認 (mecab-ipadic-NEologd は GPL)。代替として SudachiPy (Apache 2.0) を検討。

### 物流業 (運送業 SaaS) のケース

| OSS               | バージョン | ライセンス    | 区分     | 備考                                            |
| ----------------- | ---------- | ------------- | -------- | ----------------------------------------------- |
| OSRM              | -          | BSD 2-Clause  | OK       | ルーティングエンジン                            |
| OpenStreetMap データ | -       | ODbL          | 条件付き | データベースライセンス、派生物に同条件付与     |
| Leaflet.js        | 1.x        | BSD 2-Clause  | OK       | 地図表示                                        |
| MapboxGL JS       | 1.x        | BSD 3-Clause  | OK       | (2.x 以降は独自ライセンス、要再確認)           |
| GraphHopper       | -          | Apache 2.0    | OK       | -                                               |

### 出力フォーマット

SBOM の YAML サマリ形式:

```yaml
oss_license_management:
  metadata:
    project_id: "DWH-2026-Q2"
    version: "1.0"
    sbom_format: "CycloneDX 1.5"
    generated_at: "2026-04-30"
    scanner: "syft 0.95.0"
    reviewed_by:
      - role: 法務部
        name: "(氏名)"
        reviewed_at: "2026-05-01"

  policy:
    禁止ライセンス:
      - "GPL (全バージョン)"
      - "AGPL"
      - "SSPL"
      - "Commons Clause"
    要法務確認:
      - "BSL"
      - "Elastic License"
      - "ODbL"
      - "独自ライセンス"

  components:
    - name: "dbt-core"
      version: "1.7.4"
      license: "Apache-2.0"
      classification: "OK"
      source: "https://github.com/dbt-labs/dbt-core"
      obligations:
        - "NOTICE ファイル付与"
      reviewed: true

    - name: "MinIO"
      version: "RELEASE.2024-04-15T00-00-00Z"
      license: "AGPL-3.0"
      classification: "NG"
      source: "https://github.com/minio/minio"
      reason: "AGPL は SaaS 公開時に自社コード公開義務"
      remediation:
        action: "Azure Blob Storage に置換"
        deadline: "2026-08-31"
      reviewed: true

  notice_file:
    location: "https://example.com/THIRD-PARTY-NOTICES"
    last_updated: "2026-04-30"
```

## 自社コード汚染の典型シナリオ

| シナリオ                                            | 結果                                          |
| --------------------------------------------------- | --------------------------------------------- |
| 自社コードに GPL ライブラリを動的リンク             | 自社コードも GPL になる可能性 (議論あり)      |
| 自社コードに AGPL ライブラリを SaaS で利用          | 自社コードに AGPL の公開義務が発生            |
| Apache 2.0 ライブラリを利用                         | NOTICE 付与のみで OK                          |
| Web フロントで MIT ライセンスの JS を bundle        | 著作権表示を含めれば OK                       |
| Apache 2.0 ライブラリの fork し改変                 | 改変ファイルへの注記、Apache 2.0 維持で OK    |
| GPL ライブラリの fork し独自製品化                  | GPL 公開義務、独自ライセンスでの再配布不可    |

## CI/CD への組込み例

```yaml
# .github/workflows/license-check.yml
name: OSS License Check
on: [pull_request]
jobs:
  license:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate SBOM
        run: |
          curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s
          ./bin/syft scan dir:. -o cyclonedx-json > sbom.json

      - name: License Policy Check
        run: |
          # GPL/AGPL/SSPL を含むかチェック
          if jq -r '.components[].licenses[].license.id' sbom.json | grep -E 'GPL|AGPL|SSPL'; then
            echo "::error::禁止ライセンスが検出されました"
            exit 1
          fi
```

## NOTICE ファイル生成例

```markdown
# THIRD-PARTY-NOTICES

このプロジェクトは以下の OSS を利用しています。

## dbt-core 1.7.4
Copyright (c) Fishtown Analytics, Inc.
Licensed under Apache License, Version 2.0
See: https://github.com/dbt-labs/dbt-core/blob/main/License.md

## Pandas 2.2.0
Copyright (c) 2008-2024, AQR Capital Management, LLC, Lambda Foundry, Inc.
Licensed under BSD 3-Clause
See: https://github.com/pandas-dev/pandas/blob/main/LICENSE

(以下、全 OSS について同様の表記)
```

## pm-blueprint 連携

### Layer 5 リスクレジスタとの紐付け

OSS ライセンスリスクは Layer 5 リスクレジスタに登録:

```yaml
- id: R-LEG-41
  title: "AGPL ライセンス汚染による自社コード公開義務"
  category: legal
  probability: low
  impact: critical
  description: |
    SaaS 提供時に AGPL ライブラリを利用すると、
    エンドユーザーへのソース公開義務が発生し、
    競合他社にコードが公開される結果となる。
  triggers:
    - SBOM 自動チェックでの AGPL 検知
    - エンドユーザーからのソース要求
  mitigations:
    - SBOM 生成と CI でのライセンスチェック
    - 禁止ライセンスリストの社内周知
    - 代替 OSS / 商用ライブラリへの置換
  owner: "技術リード"
  kill_criterion: |
    AGPL 汚染がプロダクション環境に到達したら、
    7 営業日以内に代替実装を完了
```

```yaml
- id: R-LEG-42
  title: "OSS の特許条項違反"
  category: legal
  probability: low
  impact: severe
  description: |
    Apache 2.0 等の特許条項に違反する形 (例: 訴訟提起) で
    利用すると、ライセンスが取り消される可能性。
  mitigations:
    - 法務によるライセンス遵守状況の年次確認
    - 特許訴訟方針の事前社内合意
  owner: "法務 + 知財"
```

### 品質ゲートID対応

- **LEGAL-LICENSE-001**: 全 OSS について SBOM + 商用可否判定があれば Major 解消
- **LEGAL-AIGEN-001**: AI モデルの学習データの OSS ライセンス確認 (LAION 等)

### Layer 6 WBS への展開

- 2.5.1 SBOM ツール選定 (技術リード、3 営業日)
- 2.5.2 初期 SBOM 生成
- 2.5.3 ライセンス分類と法務レビュー (1 週間)
- 2.5.4 NG/条件付きライセンスの代替検討
- 2.5.5 CI/CD 自動チェックの組込み
- 2.5.6 NOTICE ファイル整備

## 注意事項

- ライセンス自動判定ツールは**100% ではない**。条件付きライセンスは必ず法務目視確認
- バージョンによってライセンスが変わる OSS がある (例: Elasticsearch 7.10→7.11)
- 推移的依存 (transitive dependencies) も SBOM に必ず含める
- AI モデルの学習データのライセンスも対象 (例: 著作権切れデータの利用範囲)
- "Free for personal use" は商用 NG。「商用利用許諾」の有無を必ず確認

## 参考

- IPA「OSS ライセンスの活用に関する調査」
- SPDX License List: https://spdx.org/licenses/
- CycloneDX: https://cyclonedx.org/
- OpenChain Project (ISO/IEC 5230)
- pm-blueprint Layer 7 `AI生成物著作権ポリシー.md`
- pm-blueprint Layer 5 `リスクレジスタ.md`
