---
name: 環境分離_IaC
description: dev / stg / prod を完全に分離し、Terraform を中心とした IaC ベストプラクティス、本番→stg コピー禁止などのデータフロー禁則、CI でのドリフト検知を設計するスキル。品質ゲート ENV-SEPARATE-001 / ENV-DATAFLOW-001 に対応 (Phase 2 想定)。
---

# 環境分離 + IaC ベストプラクティス

## 対応する品質ゲート (Phase 2)

| ID | 観点 | 既定の Fail/Conditional 条件 |
|----|------|------|
| ENV-SEPARATE-001 | 環境完全分離 | dev / stg / prod が同一サブスクリプション内、同一テナントなど分離不十分 |
| ENV-DATAFLOW-001 | データフロー禁則 | 本番 → stg コピー / 本番 PII の検証利用 など禁則ルール未宣言 |

## 概要

「環境を分けています」と言いながら、実態は:

- 同じ AWS アカウントで Tag だけで区別
- 本番の DB スナップショットを stg に直接コピーして検証
- prod の Service Principal が dev も操作できる

…というプロジェクトは多い。これは「分離していない」の典型例。

本スキルでは:

1. **5 層の分離レベル** (Identity / Network / Data / Compute / Pipeline) を全層独立化
2. **Terraform IaC** で構成を再現可能に
3. **データフロー禁則** を明文化
4. **CI でドリフト検知** を毎日自動実行

を設計する。

## いつ使うか

- 監査 (J-SOX, ISMS, ISO 27001) を見据える時
- 本番事故の RCA で「stg と本番が違って見落とした」が出た時
- セキュリティ監査で「本番データが検証で使われている」と指摘された時
- 複数チームが並行開発で互いの環境を壊し始めた時

## 5 層の分離レベル (ENV-SEPARATE-001)

### 層 1: Identity / Tenant 分離

| 分離度 | 構成 | 推奨度 |
|-------|------|------|
| 弱 | 同テナント、同サブスクリプション、Tag で区別 | × 採用不可 |
| 中 | 同テナント、サブスクリプション分離 | △ 検証期のみ |
| **強** | **テナント分離 + サブスクリプション分離** | ◎ **推奨** |

### 層 2: Network 分離

```yaml
network_isolation:
  prod:
    vnet: vnet-prod-japaneast (10.10.0.0/16)
    subscription: sub-prod
    peering: stg ✗, dev ✗  # 直接接続禁止
  stg:
    vnet: vnet-stg-japaneast (10.20.0.0/16)
    subscription: sub-stg
  dev:
    vnet: vnet-dev-japaneast (10.30.0.0/16)
    subscription: sub-dev
  shared:
    egress: prod 専用 NAT
    ingress: WAF 別構成
```

### 層 3: Data 分離

| データ種別 | dev | stg | prod | 備考 |
|-----------|-----|-----|------|------|
| 顧客 PII | **禁止** | 合成データ | 実データ | dev/stg では絶対に使わない |
| 通話録音 | **禁止** | 同意取得済み少量 | 実データ | WORM Storage |
| マスタデータ | サンプル | 本番スナップ + マスキング | 実データ | 月次スナップ |
| 構成パラメータ | dev 値 | 本番相当 | 本番値 | Key Vault 別建て |

### 層 4: Compute 分離

```yaml
compute_isolation:
  managed_identity:
    prod: mi-app-prod (権限: prod-vault, prod-storage のみ)
    stg: mi-app-stg
    dev: mi-app-dev
  service_principal:
    prod_deploy_sp: prod デプロイ専用、CI 専用
    cross_env_sp: 禁止
  rbac:
    prod_admin_role: 4 名のみ、JIT (Privileged Identity Mgmt)
    stg/dev_admin: チームメンバー (常時)
```

### 層 5: Pipeline 分離

```yaml
pipeline_isolation:
  cicd:
    prod_branch: main (Protected, Required Reviewers 2)
    stg_branch: release/*
    dev_branch: feature/*
  approval_gates:
    dev → stg: 自動 (テストパス)
    stg → prod: 手動承認 + Change Advisory Board
  artifact_registry:
    prod: acr-prod (Immutable Tag)
    stg: acr-stg
    dev: acr-dev
```

## Terraform IaC ベストプラクティス

### 推奨ディレクトリ構造

```
infra/
├── modules/                    # 再利用モジュール (環境非依存)
│   ├── postgres-flex/
│   ├── container-apps-env/
│   ├── databricks-workspace/
│   ├── eventhub-namespace/
│   └── keyvault/
├── envs/                       # 環境別 root モジュール
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf          # state は dev 専用 storage
│   ├── stg/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── terraform.tfvars
│       └── backend.tf          # state は prod 専用 storage
└── shared/                     # クロス環境のリソース (例: DNS, ACM)
    └── dns/
```

### State 分離

```hcl
# infra/envs/prod/backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-tfstate-prod"
    storage_account_name = "sttfstateprod001"
    container_name       = "tfstate"
    key                  = "app/prod.tfstate"
    use_oidc             = true        # GitHub Actions OIDC
  }
}
```

**鉄則:**

- **state ファイルは環境ごとに完全分離**
- **state の保管 storage は本体リソースとは別 RG**
- **state には KMS 暗号化、versioning, locking 必須**
- **dev/stg state には prod の secret を絶対書かない**

### モジュール設計の原則

```hcl
# infra/modules/postgres-flex/variables.tf
variable "environment" {
  type        = string
  description = "dev / stg / prod"
  validation {
    condition     = contains(["dev", "stg", "prod"], var.environment)
    error_message = "environment は dev/stg/prod のいずれか"
  }
}

variable "high_availability" {
  type    = bool
  default = false  # prod のみ true 想定
}

variable "backup_retention_days" {
  type    = number
  default = 7
}

# infra/envs/prod/main.tf
module "postgres" {
  source                 = "../../modules/postgres-flex"
  environment            = "prod"
  high_availability      = true
  backup_retention_days  = 35
  geo_redundant_backup   = true
}
```

### CI / CD ベストプラクティス

```yaml
# .github/workflows/terraform.yml
name: terraform
on:
  pull_request:
    paths: [infra/**]
  push:
    branches: [main]

jobs:
  plan_dev:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID_DEV }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID_DEV }}
      - run: terraform -chdir=infra/envs/dev init
      - run: terraform -chdir=infra/envs/dev plan -out=tfplan
      - run: terraform -chdir=infra/envs/dev show -json tfplan > tfplan.json
      - uses: bridgecrewio/checkov-action@v12
        with:
          file: tfplan.json
          quiet: true

  apply_prod:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production   # 手動承認ゲート
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID_PROD }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID_PROD }}
      - run: terraform -chdir=infra/envs/prod apply -auto-approve
```

### Secret 管理

| Secret 種別 | 保管先 | 取得方法 |
|-----------|------|--------|
| DB パスワード | Azure Key Vault (env 別) | Managed Identity |
| API キー | Key Vault | Managed Identity |
| Terraform 自身の SP 認証 | OIDC (GitHub Actions ↔ Azure AD) | GitHub OIDC Token |
| 開発者の手動操作用 | なし (PIM JIT) | Just-In-Time Elevation |

**鉄則:**

- **Secret を tfvars に書かない**。`tfvars` は Git に上げ、Secret は KV から `data "azurerm_key_vault_secret"` で取得
- **`terraform output` で Secret を出力しない** (state に平文保存される)
- **CI 環境変数に Secret を直書きしない** (OIDC 採用)

## データフロー禁則 (ENV-DATAFLOW-001)

### 禁則ルール 5 つ (合言葉化)

```
1. 本番から下流環境への実データコピー禁止 (prod → stg/dev)
2. PII / 録音 / 通話履歴の dev 利用禁止
3. 環境を跨ぐ Service Principal / 認証情報共有禁止
4. 環境間の VNet ピアリング禁止
5. 本番 Read-Only であっても dev/stg からの直接接続禁止
```

### 例外手続き (やむを得ない場合)

```yaml
exception_process:
  approval_chain:
    - データガバナンス責任者
    - セキュリティ責任者
    - プライバシー責任者 (PII の場合)
  conditions_required:
    - PII マスキング (氏名 / 電話 / 住所 / メール / 録音)
    - 期間限定 (最大 7 日)
    - 利用後の確実な削除
    - 監査ログ完備
  documentation:
    - 例外申請書 (Issue 化)
    - 削除証跡 (CRC32 ログ)
    - 監査委員会 月次報告
```

### 合成データ生成方針 (推奨)

| 用途 | 推奨手段 |
|-----|--------|
| 顧客マスタ | Faker + ドメインルール (郵便番号妥当性) |
| 通話録音 | TTS で合成、業務シナリオを 30 件程度 |
| 通話テキスト | LLM 合成 (Azure OpenAI), 業務マニュアル仮説主軸 |
| 取引データ | スキーマ準拠の乱数 + 季節性関数 |

(参考: ja_domain_poc_synthetic.md - 公開英語データの翻訳だけでは業務実態と乖離するため、業務マニュアル仮説主軸が必要)

### マスキング規則

```yaml
masking_rules:
  pii_columns:
    - {column: name,    rule: "Hash(SHA-256)[:8] + 'さん'"}
    - {column: phone,   rule: "0X0-XXXX-####"}
    - {column: email,   rule: "user_<id>@example.invalid"}
    - {column: address, rule: "都道府県のみ残す"}
  voice:
    - {file: wav, rule: "TTS で別音声に置換 or 完全削除"}
  transcript:
    - {column: text, rule: "PII 検出 → [MASKED] 置換"}
  validation:
    - 再識別テストを月次実行 (旧 ID 復元できないか)
```

## ドリフト検知 CI (毎日自動)

### 仕組み

```
                      [GitHub Actions Scheduled (毎日 03:00 JST)]
                             ↓
                      terraform plan -refresh-only
                             ↓
                      差分あり?
                       /        \
                     なし        あり
                      ↓           ↓
                    OK    Slack/Teams 通知
                          + Issue 起票
                          + 4h 以内に対応必須
```

### 実装例

```yaml
# .github/workflows/drift-detection.yml
name: drift-detection
on:
  schedule:
    - cron: '0 18 * * *'   # UTC 18:00 = JST 03:00
  workflow_dispatch:

jobs:
  detect:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        env: [dev, stg, prod]
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v1
        with:
          client-id: ${{ secrets[format('AZURE_CLIENT_ID_{0}', matrix.env)] }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets[format('AZURE_SUBSCRIPTION_ID_{0}', matrix.env)] }}
      - id: plan
        run: |
          cd infra/envs/${{ matrix.env }}
          terraform init
          terraform plan -refresh-only -detailed-exitcode -out=plan.bin || echo "exitcode=$?" >> $GITHUB_OUTPUT
      - if: steps.plan.outputs.exitcode == '2'
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          text: |
            🚨 Drift Detected (${{ matrix.env }})
            terraform plan -refresh-only に差分が検出されました。
            手動変更の有無を 4 時間以内に確認してください。
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### ドリフト発生時の RCA テンプレート

```yaml
drift_rca:
  detected_at: 2026-05-15 03:00 JST
  environment: prod
  resource: azurerm_postgresql_flexible_server.main
  attribute_changed: [sku_name, backup_retention_days]
  suspected_change_method:
    - Azure Portal 手動変更
    - 別の SP からの直接 API 操作
  investigation:
    - Azure Activity Log を確認 (caller, time)
    - PIM の有効化履歴を確認
  remediation:
    - 手動変更を Terraform で再現 or revert
    - PR で Apply
  prevention:
    - 該当 RG に Deny Policy 追加
    - PIM 必須化、JIT 有効化
```

## 出力フォーマット

```
docs/
├── environment/
│   ├── 01_isolation_layers.md      # ENV-SEPARATE-001
│   ├── 02_dataflow_rules.md        # ENV-DATAFLOW-001
│   ├── 03_iac_guidelines.md
│   └── 04_drift_detection.md
infra/
├── modules/
└── envs/{dev,stg,prod}/
.github/workflows/
├── terraform.yml
└── drift-detection.yml
```

## 例示: コールセンター VoC 基盤の環境分離

```yaml
example_environments:
  prod:
    subscription: sub-vop-prod (専用)
    network: vnet-prod 10.10.0.0/16
    data:
      transcript: 実データ (Genesys Cloud → Postgres)
      pii: 実データ (Customer マスタ)
    access:
      admin: 4 名 + PIM JIT
      deploy_sp: GitHub OIDC のみ
  stg:
    subscription: sub-vop-stg
    network: vnet-stg 10.20.0.0/16
    data:
      transcript: 合成 + 同意取得済み少量 (50 件)
      pii: マスキング済み
  dev:
    subscription: sub-vop-dev
    network: vnet-dev 10.30.0.0/16
    data:
      transcript: 完全合成 (LLM 生成 100 件)
      pii: 完全合成
forbidden_flows:
  - prod_db -> stg_db (ANY)
  - prod_blob -> dev_blob (ANY)
  - cross_env_sp_sharing
  - vnet_peering between envs
```

## 作成時のチェックリスト

- [ ] 5 層 (Identity/Network/Data/Compute/Pipeline) すべて分離設計済み
- [ ] サブスクリプション (Account) が環境ごとに分離されている
- [ ] VNet が環境ごとに独立、ピアリング禁止が明文化されている
- [ ] state ファイルが環境別 storage で完全分離
- [ ] Secret は Key Vault に保管され tfvars / state にない
- [ ] CI が GitHub OIDC + 環境別 SP で実行される
- [ ] prod 適用は手動承認ゲートを通る
- [ ] 5 つの禁則ルールが文書化され、合言葉化されている
- [ ] PII / 通話録音の dev 利用禁止が明文化されている
- [ ] 例外手続きが定義され、承認チェーンが明確
- [ ] 合成データ生成方針があり、業務マニュアル仮説に基づく
- [ ] ドリフト検知 CI が毎日自動実行される
- [ ] ドリフト発生時の RCA テンプレートがある

## pm-blueprint 連携

| 既存サブスキル | 連携ポイント |
|--------------|-----------|
| `layer-3-architecture/アーキスタイル選定.md` | Hosting Tier 選定後にこの環境分離設計を発動 |
| `layer-3-architecture/ADR作成.md` | 環境分離方針 / IaC ツール選定は ADR (Type 1) |
| `layer-3-architecture/データ設計詳細.md` | データフロー禁則は SoT 宣言と一体運用 |
| `layer-4-requirements/RTO_RPO_DR設計.md` | リージョン冗長 (Japan East/West) は本スキルの環境分離設計に追加 |
| `layer-4-requirements/可観測性三本柱.md` | 各環境ログを別 Workspace に分離 |

## 品質ゲート対応サマリ

| ゲート ID | 本スキル該当節 | 出力アーティファクト |
|----------|--------------|------------------|
| ENV-SEPARATE-001 | 5 層分離 + IaC | `docs/environment/01_isolation_layers.md` + ADR |
| ENV-DATAFLOW-001 | データフロー禁則 | `docs/environment/02_dataflow_rules.md` |

## 参考

- HashiCorp, "Terraform Best Practices" (https://www.terraform-best-practices.com/)
- Microsoft Cloud Adoption Framework: Landing Zones
- Azure Verified Modules (AVM): https://aka.ms/avm
- AWS Multi-Account Strategy
- Bridgecrew Checkov, tfsec (Static Analysis)
- ISO/IEC 27001, ISMS 環境分離要件
- 個人情報保護法ガイドライン (検証データの取扱)
- (社内) feedback_ja_domain_poc_synthetic.md (合成データ設計)
