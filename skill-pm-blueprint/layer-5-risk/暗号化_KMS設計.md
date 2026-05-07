---
name: 暗号化_KMS設計
description: 保存時/転送時暗号化、Azure Key Vault を中心とした KMS 設計、鍵ローテーション運用、BYOK・Envelope Encryption・HSM 採否を体系化するスキル。SEC-CRYPTO-001 対応。
---

# 暗号化_KMS設計

> **品質ゲート対応**: SEC-CRYPTO-001 (暗号化方針が「TLS 1.3 / 保存時暗号化」程度の抽象記述に留まり、鍵管理・ローテーション・BYOK 採否が未設計)

## 概要

データ分析基盤・コールセンター基盤では、PII・録音音声・文字起こし・LLM プロンプト/応答など機微情報の集積点が多く、暗号化は単なる「TLS と保存時暗号化」では足りない。**何を・どの粒度で・どの鍵で・誰が・いつまで** 暗号化するかを設計し、鍵管理サービス (KMS) の責務を明確にする必要がある。

主要な設計領域:

1. **転送時暗号化** (TLS 1.2/1.3, mTLS, IPsec, Private Link)
2. **保存時暗号化** (ストレージ層 / DB 層 / アプリ層 / フィールド層)
3. **鍵管理** (KMS = Azure Key Vault, AWS KMS, GCP Cloud KMS)
4. **鍵ローテーション** (自動/手動、年/四半期、緊急時)
5. **BYOK / HYOK / CMK** (顧客が鍵を所持するか)
6. **Envelope Encryption** (DEK と KEK の階層化)
7. **HSM** (Hardware Security Module、FIPS 140-2/3 Level 認定)

## いつ使うか

- Layer 3 アーキテクチャでデータレイク/DWH (Databricks, SAP) の構成が決まったとき
- Layer 5 STRIDE で「Information disclosure (I)」「Tampering (T)」が検出されたとき
- 録音 5-10 年保管要件が確定したとき (鍵ライフサイクル設計が必要)
- LLM 利用が決まったとき (プロンプト/応答の暗号化方針)
- 監査・規制対応 (PCI DSS, FISC 安全対策基準, ISO27001 A.10) が必要なとき
- マルチクラウド/SaaS 連携で鍵の所在問題が発生するとき

## 手順

### ステップ 1: 暗号化対象のデータ分類

データを 4 分類し、暗号化方針を決める:

| 分類 | 例 | 転送時 | 保存時 | フィールド/列 |
|-----|-----|--------|--------|--------------|
| **PII (氏名・住所・電話)** | 会員 DB、CRM | TLS 1.3 必須 | CMK 暗号化 | 列暗号化 + マスキング |
| **音声・映像** | コール録音、ビデオ通話 | TLS 1.3 + DRM | CMK + Envelope | ファイル単位 KEK |
| **文字起こし・LLM 入出力** | Transcripts, Prompt Log | TLS 1.3 | CMK 暗号化 | 機微部のみフィールド暗号化 |
| **集計 KPI・公開情報** | ダッシュボード、月次レポート | TLS 1.3 | プラットフォーム既定暗号化 | 不要 |

### ステップ 2: 転送時暗号化の設計

| 区間 | 推奨 | 注意点 |
|-----|------|--------|
| ユーザ ⇔ Web/BI | TLS 1.3, HSTS, CSP | TLS 1.0/1.1 を明示拒否 |
| サービス間 (内部) | mTLS (相互認証) | 「内部だから暗号化不要」は撤廃 |
| クラウド ⇔ オンプレ | IPsec VPN または ExpressRoute + Private Endpoint | パブリックエンドポイント禁止 |
| LLM API 連携 | TLS 1.3 + Private Link / Customer-managed Endpoint | API キーは Key Vault 管理 |
| データソース (POS, SAP) ⇔ Ingest | TLS 1.3 + サービス専用 ID | 自己署名証明書を本番で許可しない |

**証明書管理**: Azure Key Vault Certificate もしくは Let's Encrypt + cert-manager。期限切れアラートを 60 日前から発報。

### ステップ 3: 保存時暗号化の階層

```
┌──────────────────────────────────────────────┐
│ アプリ層暗号化 (Field-Level Encryption)      │
│   例: SSN, クレジットカード番号               │
└──────────────────────────────────────────────┘
              ▲ DEK (フィールド単位)
┌──────────────────────────────────────────────┐
│ DB / プラットフォーム暗号化 (TDE / Unity Cat.)│
│   例: Databricks 顧客管理キー、SAP HANA TDE  │
└──────────────────────────────────────────────┘
              ▲ DEK (テーブル/DB 単位)
┌──────────────────────────────────────────────┐
│ ストレージ層暗号化 (Azure Storage CMK / SSE) │
│   例: Blob, ADLS Gen2, Disk                  │
└──────────────────────────────────────────────┘
              ▲ KEK (ストレージアカウント単位)
┌──────────────────────────────────────────────┐
│ Azure Key Vault (Premium / Managed HSM)      │
│   - KEK 保管・ラップ/アンラップ              │
│   - アクセスポリシー = Entra ID + RBAC       │
└──────────────────────────────────────────────┘
```

**Envelope Encryption の流れ**:
1. データを DEK (Data Encryption Key, 256bit AES) で暗号化
2. DEK を KEK (Key Encryption Key, Key Vault 内) でラップ
3. ラップ済 DEK をデータと並べて保管
4. 復号時に KEK で DEK をアンラップ → データ復号

利点: KEK ローテーションは「ラップ済 DEK の再暗号化」だけで済み、データ本体を再暗号化しなくてよい。

### ステップ 4: Azure Key Vault を中心とした KMS 設計

```
┌─────────────────────────────────────────────────┐
│ Azure Key Vault Premium / Managed HSM            │
│                                                  │
│  RG: rg-vault-prod  (本番環境専用)               │
│   ├─ Vault: kv-voc-prod                          │
│   │   ├─ Key: kek-storage-blob (RSA-HSM 4096)    │
│   │   ├─ Key: kek-databricks-uc (RSA-HSM 4096)   │
│   │   ├─ Key: kek-sap-hana (RSA-HSM 4096)        │
│   │   ├─ Secret: genesys-api-token (90日RT)      │
│   │   └─ Certificate: api.example.com (1年RT)    │
│                                                  │
│  Access Control:                                 │
│   - Key Vault Administrator: SREチーム (PIM要)   │
│   - Key Vault Crypto User: 各サービス MI          │
│   - Network: Private Endpoint, Firewall ON       │
│   - Logging: Diagnostics → Log Analytics + 90日  │
└─────────────────────────────────────────────────┘
```

設計の必須要件:

- **Premium または Managed HSM**: HSM-protected key を使用 (Standard では Software-protected)。FIPS 140-2 Level 2/3 認定。
- **Soft Delete + Purge Protection**: 誤削除/悪意削除からの保護を有効化、Purge Protection は変更不可フラグ。
- **Private Endpoint**: パブリックネットワークアクセスは Disabled。VNet からのみアクセス。
- **Diagnostics Logs**: Microsoft Sentinel/Purview に転送し、`KeyVault Operations` (KeyGet, KeyWrap 等) を全件追跡。
- **RBAC**: Key Vault Access Policy ではなく **Azure RBAC ベース** を採用 (Entra ID と統合、PIM 対応)。
- **マルチリージョン**: 主要リージョン + フェイルオーバー先を Replication-enabled な Managed HSM で構成。

### ステップ 5: 鍵ローテーション運用

| 鍵種別 | ローテーション周期 | 方式 | 緊急時 |
|-------|------------------|------|--------|
| KEK (RSA 4096) | 1 年 | Key Vault 自動ローテーションポリシー | Compromise 時即時手動 |
| DEK (AES 256) | 暗号化対象更新時 | Envelope 再ラップで KEK 切替に追従 | KEK 同様 |
| API シークレット | 90 日 | Key Vault Rotation Function | 即時 |
| TLS 証明書 | 1 年 | Key Vault Certificate 自動更新 (Let's Encrypt 連携可) | 即時 |
| 開発環境鍵 | 6 ヶ月 | 同上 | 同上 |

**自動ローテーションポリシー (Key Vault) サンプル**:

```json
{
  "lifetimeActions": [
    {
      "trigger": { "timeAfterCreate": "P11M" },
      "action": { "type": "Rotate" }
    },
    {
      "trigger": { "timeBeforeExpiry": "P30D" },
      "action": { "type": "Notify" }
    }
  ],
  "attributes": {
    "expiryTime": "P1Y"
  }
}
```

**運用ルール**:

1. ローテーション直後 1 週間は旧鍵を `Disabled` にせず保持 (ロールバック窓)
2. 鍵の世代番号 (`/keys/<name>/<version>`) を監査ログで追跡し、復号失敗をアラート化
3. 「年次」の鍵は四半期に "ヘルスチェック" (擬似ローテーションのドライラン) を実施
4. 鍵ローテーションランブックを CISO + SRE で四半期レビュー

### ステップ 6: BYOK / HYOK / CMK 採否判断

| モデル | 鍵生成 | 鍵保管 | クラウド使用 | 採否判断 |
|-------|-------|--------|-------------|---------|
| **CMK** (Customer-Managed Key) | クラウド KMS | クラウド KMS | クラウド KMS 内で使用 | デフォルト推奨 |
| **BYOK** (Bring Your Own Key) | 顧客 HSM | クラウド KMS にインポート | クラウド KMS 内で使用 | 規制で求められる場合 |
| **HYOK** (Hold Your Own Key) | 顧客 HSM | 顧客側のみ | 顧客 HSM で使用 | 強い規制 (一部金融, 政府) |
| **Confidential Computing** | クラウド | クラウド+TEE | TEE 内で復号 | 復号データを誰にも見せない要件 |

**採否決定フローチャート**:

```
規制で「鍵の自社管理」が明文化されているか?
  Yes → BYOK 検討
   ├─ 鍵をクラウドに渡せるか?
   │   Yes → BYOK 採用 (Azure Key Vault Premium にインポート)
   │   No  → HYOK 採用 (オンプレ HSM, Azure Confidential Computing 検討)
  No  → CMK 採用 (Azure Key Vault Premium / Managed HSM)
```

**コールセンター/データ基盤の典型解**:

- 一般顧客向け SaaS: CMK で十分 (運用負荷とのバランス)
- 金融機関 PoC・公共: BYOK (規制要件 + 監査説明性)
- 国家機密級: HYOK + Confidential Computing (本案件は通常該当しない)

### ステップ 7: HSM 活用判断

| 観点 | Software-protected (Standard) | HSM-protected (Premium) | Managed HSM (Single-Tenant) |
|------|-------------------------------|------------------------|-----------------------------|
| FIPS 140-2 認定 | Level 1 | Level 2 | Level 3 |
| 単一テナント | No | No | Yes |
| 運用負荷 | 低 | 低 | 中 |
| 価格 (月額目安) | 数千円〜 | 数万円〜 | 十数万円〜 |
| 推奨ケース | 開発/検証 | 一般本番 (PII) | 金融, 公共, 大規模 PoC |

判断基準:

- **PII を扱う本番**: 最低 Premium (HSM-protected)
- **録音 5-10 年保管 + 監査要件**: Managed HSM 推奨 (FIPS 140-2 L3)
- **マルチテナント禁止条項のある契約**: Managed HSM 必須

### ステップ 8: ADR テンプレート (`docs/adr/ADR-xxx-encryption-kms.md`)

```markdown
# ADR-xxx 暗号化と KMS 設計

## ステータス
Accepted (YYYY-MM-DD) / Reviewer: <CISO 名>, <CIO 名>

## コンテキスト
- 対象データ: PII, 音声録音 (5-10年), 文字起こし, LLM 入出力
- 規制: 個人情報保護法, FISC 安全対策基準 (該当する場合)
- 既存環境: Azure (Databricks + ADLS Gen2 + SAP on Azure)

## 決定
- 転送時: TLS 1.3 必須, 内部間も mTLS
- 保存時: Envelope Encryption (DEK/KEK 階層)
- KMS: Azure Key Vault Premium + 一部 Managed HSM
- 鍵管理: CMK (BYOK は次フェーズで再評価)
- ローテーション: KEK=1年, シークレット=90日, 証明書=1年
- HSM: HSM-protected key (Premium) を全本番 KEK に適用

## 検討した代替案
1. Standard Vault (Software key) → FIPS L1 のため却下
2. HYOK (オンプレ HSM のみ) → クラウド利点が消えるため却下
3. ローテーション 3 年 → 業界標準 1 年に劣後

## 帰結
- ポジティブ: 規制適合, 鍵ライフサイクル明確化
- ネガティブ: Premium 月額追加コスト
- 中立: 初期構築 1 人月

## 関連
- 品質ゲート: SEC-CRYPTO-001
- 関連 ADR: 認可方式選定, ゼロトラスト方針
- 関連リスク: R-CRYPT-001 (鍵漏洩), R-CRYPT-002 (鍵紛失で復号不能)
```

## 出力フォーマット

1. **データ分類×暗号化マトリクス** (転送時/保存時/フィールド)
2. **KMS 構成図** (Azure Key Vault, Private Endpoint, RBAC, Diagnostics)
3. **鍵ライフサイクル表** (鍵種別×ローテーション×保管期間)
4. **CMK/BYOK/HYOK 判定結果**
5. **HSM 採否判定結果**
6. **ADR 1 本**

## データ分析基盤・コールセンター向け実例ノート

- **録音 5-10 年保管**: 古い録音の復号鍵が消えるとデータ自体が無価値になる。**KEK は永久に Soft Delete + Purge Protection、世代を物理的にアーカイブ**。Envelope Encryption で「ラップ済 DEK」をデータと一緒に長期保管。
- **Databricks Unity Catalog**: ストレージは ADLS Gen2 + Customer-managed Key。Databricks ワークスペースのコントロールプレーン暗号化も Key Vault 連携 (Customer-managed keys for managed services)。
- **SAP on Azure**: SAP HANA TDE + Azure Disk Encryption の二重化。Key Vault の KEK を SAP 用ボールトとして分離。
- **Genesys 録音**: ベンダー側暗号化 + 顧客側でダウンロード後再暗号化 (Envelope) 。ベンダー漏洩時の二段防御。
- **LLM 利用案件**: プロンプト/応答ログを保管する場合、PII 含有部はフィールド暗号化。ベクトル DB (埋め込み) も "再構成可能" な PII になり得るため暗号化対象。
- **コールセンタ ASP の API キー**: Key Vault Secret + 90 日ローテーション + Rotation Function を Genesys API ↔ AppRegistration の両側で同期。

## STRIDE との関係

- STRIDE の「Information disclosure (I)」「Tampering (T)」「Spoofing (S)」に対する**実装手段**として本スキルが詳細を埋める。
- STRIDE は「何が脅威か」、本スキルは「どの暗号と鍵で守るか」。
- リスクレジスタには `linked_threat_id: TM-I-002` `linked_kms_decision: ADR-xxx` で相互参照。

## pm-blueprint 連携

- **Layer 5 既存サブスキル**:
  - 脅威モデリング.md: STRIDE で I/T/S 脅威に紐付き、本スキルが実装方針を提供。
  - リスクレジスタ.md: 鍵関連リスク (R-CRYPT-xxx) を本スキルの ADR と紐付ける。
- **Layer 6 (新規) ゼロトラスト方針**: 「内部は安全」幻想撤廃と整合し、内部間 mTLS + フィールド暗号化を強制。
- **Layer 7 法務・規制**: 個人情報保護法 / FISC / 業界規制を ADR の "コンテキスト" 節で明示。
- **Layer 8 LLM ガバナンス**: LLM プロンプト/応答の暗号化と鍵分離を本スキルで規定。
- **Layer 9 運用**: 鍵ローテーション、緊急時ランブック、監査ログレビューを運用設計に組み込む。
- **品質ゲート対応**:
  - SEC-CRYPTO-001 (本スキル直接対応)
  - SEC-AUTHZ-DESIGN-001 (Key Vault RBAC は認可方式選定と整合)
  - SEC-ZEROTRUST-001 (内部間 mTLS は ZTA 実装手段)
  - LEGAL-THREATMODEL-001 (LINDDUN の Disclosure 観点で暗号化粒度の根拠)

## 参考

- NIST SP 800-57 "Recommendation for Key Management"
- Microsoft Learn "Azure Key Vault Premium / Managed HSM"
- FIPS PUB 140-2 / 140-3 "Security Requirements for Cryptographic Modules"
- ISO/IEC 27002 A.10 "Cryptographic Controls"
- 個人情報保護委員会「個人データの安全管理措置に関するガイドライン」
- FISC「金融機関等コンピュータシステムの安全対策基準」
