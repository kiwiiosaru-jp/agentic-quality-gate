---
name: PII境界_DLP
description: LLM 入出力における PII (個人識別情報) の境界制御。DLP 検出、PII マスキング (regex / NER / GiNZA-spaCy / Presidio)、Azure OpenAI 学習オプトアウト契約、海外移転対応。LLM-PII-001 / OWASP LLM02 (Sensitive Information Disclosure) 対応。
---

# PII境界 / DLP

## 対応する品質評価ID

- **LLM-PII-001** (Fail → Pass): LLM 入出力での PII 漏洩リスクと DLP 体制の不足
- **OWASP LLM02** (Sensitive Information Disclosure): 機微情報開示
- **OWASP LLM06** (Excessive Agency): エージェントが PII を過剰送信
- **GOV-AI-DATAFLOW-001** (Conditional → Pass): AI データフローの全体ガバナンス

## 概要

LLM プロンプトに PII (個人情報) が混入することは、個人情報保護法・GDPR・業種別規制 (金融・医療) の重大な抵触リスクである。本スキルは以下を統合して**入力経路・処理経路・出力経路**の三層で PII を防護する:

1. **DLP 検出** : 入出力ストリームでの PII 自動検知
2. **PII マスキング** : regex + NER (固有表現抽出) + spaCy / GiNZA / Presidio
3. **Azure OpenAI 学習オプトアウト契約** : ベンダー側の利用条項確認
4. **海外移転対応** : リージョン制約と契約根拠の文書化
5. **トークン化** : 可逆マスキング (decrypt 鍵を別管理) によるユーザー復元

## いつ使うか

- LLM 入力に顧客名・住所・電話・メール・カード番号・健康情報が含まれうる場合
- コールセンター録音 → STT → LLM 経路 (UC-003)
- メール本文を LLM が処理する場合 (UC-008)
- 業務 DB の Bronze/Silver 層を LLM が触る場合
- 外部ベンダー LLM (OpenAI, Anthropic, Google) を利用する場合
- 海外リージョン (US/EU) の LLM API を呼ぶ場合

## 手順

### ステップ 1: PII 種別の定義

国内法 (個人情報保護法 2022 改正) に基づく分類:

| 機微度 | 種別 | 例 |
|---|---|---|
| 要配慮個人情報 | 病歴・犯歴・人種・信条 | 通院記録、刑事処分歴 |
| 個人識別符号 | マイナンバー、運転免許証番号、パスポート番号 | 住基カード番号 |
| 個人情報 (狭義) | 氏名、住所、電話、メール、生年月日、顔画像 | 顧客マスタ |
| 準個人情報 | 単独では識別不可だが組合せで識別可能 | 都道府県+性別+年齢層 |
| カードホルダーデータ (PCI DSS) | カード番号、CVV、有効期限 | 決済情報 |

各種別ごとに**LLM 送信可否**を明示する。

| 種別 | LLM 送信可否 | マスキング必要 | 備考 |
|---|---|---|---|
| 要配慮個人情報 | NG | - | 絶対に LLM へ送らない |
| マイナンバー | NG | - | 法令で目的外利用禁止 |
| カード番号 | NG | - | PCI DSS 違反 |
| 氏名 | 条件付 | 必須 (トークン化) | UC-003 で必要なら可逆マスキング |
| 住所 | 条件付 | 必須 (郵便番号レベルまで丸める) | |
| 電話 | 条件付 | 必須 (下 4 桁マスク) | |
| メール | 条件付 | 必須 (ローカル部マスク) | |
| 顔画像 | NG | - | LLM 送信不可 |

### ステップ 2: DLP 検出ルールの整備

#### 2-1. Regex 検出

```yaml
# dlp_rules_regex.yaml
rules:
  - id: PII-PHONE-JP
    name: 日本の電話番号
    pattern: '\b0\d{1,4}[-\s]?\d{1,4}[-\s]?\d{4}\b'
    severity: high
  - id: PII-EMAIL
    name: メールアドレス
    pattern: '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
    severity: high
  - id: PII-MYNUMBER
    name: マイナンバー (12 桁)
    pattern: '\b\d{4}\s?\d{4}\s?\d{4}\b'
    severity: critical
  - id: PII-CREDIT-CARD
    name: クレジットカード番号 (Luhn)
    pattern: '\b(?:4\d{3}|5[1-5]\d{2}|3[47]\d{2}|6011)[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b'
    severity: critical
    extra_check: luhn_validation
  - id: PII-POSTAL-JP
    name: 郵便番号
    pattern: '〒?\d{3}[-\s]?\d{4}'
    severity: medium
  - id: PII-PASSPORT-JP
    name: パスポート番号 (10 桁: 英字 2 + 数字 7)
    pattern: '\b[A-Z]{2}\d{7}\b'
    severity: critical
```

#### 2-2. NER (固有表現抽出) 検出

regex で取り切れない**氏名**や**地名**は NER で検出する。

| エンジン | 言語 | ライセンス |
|---|---|---|
| spaCy + GiNZA | 日本語 | MIT |
| Microsoft Presidio | 多言語 | MIT |
| Azure AI Language - PII Detection | 日英他 | 商用 |
| AWS Comprehend - PII Detection | 多言語 | 商用 |

```python
import spacy
import ginza  # 日本語 NLP

nlp = spacy.load("ja_ginza")
doc = nlp(user_input)
detected_pii = [
    {"text": ent.text, "label": ent.label_, "start": ent.start_char, "end": ent.end_char}
    for ent in doc.ents
    if ent.label_ in ["Person", "Location", "Organization"]
]
```

#### 2-3. Microsoft Presidio (推奨)

```python
from presidio_analyzer import AnalyzerEngine
from presidio_anonymizer import AnonymizerEngine

analyzer = AnalyzerEngine(supported_languages=["ja", "en"])
results = analyzer.analyze(text=user_input, language="ja")
anonymizer = AnonymizerEngine()
masked = anonymizer.anonymize(text=user_input, analyzer_results=results)
# masked.text に [PERSON_001] [PHONE_NUMBER_001] 等で置換された結果
```

### ステップ 3: マスキング戦略

| 戦略 | 説明 | 用途 |
|---|---|---|
| Redaction (削除) | `***` に置換 | 後続処理で復元不要なケース |
| Generalization (汎化) | 「東京都渋谷区」→「東京都」 | 統計分析用 |
| Tokenization (トークン化) | 別 DB に保管した ID で置換 | UC-003 で復元必要 |
| Pseudonymization (仮名化) | 「田中太郎」→「PERSON_001」 | RAG 検索用 |
| Encryption (暗号化) | KMS 鍵で encrypt | バックアップ・アーカイブ |

#### LLM 送信時の標準フロー

```
[ユーザー入力]
    ↓
[DLP 検出] (regex + NER + Presidio)
    ↓
[トークン化] (PII → ID 置換、ID ↔ 原文を Vault に保管)
    ↓
[LLM 呼び出し]  ← 送信前に最終チェック (PII 残存検知)
    ↓
[LLM 出力]
    ↓
[出力 DLP] (LLM が PII を漏洩していないか検査)
    ↓
[トークン展開] (ユーザー向け表示時のみ Vault から復元)
    ↓
[ユーザー表示]
```

### ステップ 4: Azure OpenAI 学習オプトアウト契約

外部 LLM ベンダーへ送信する PII は**ベンダーの学習に使われない**ことを契約で確保する。

| ベンダー | 既定 | オプトアウト方法 |
|---|---|---|
| Azure OpenAI Service | デフォルト学習に**使用しない** (Microsoft の公式立場) | Data Privacy Addendum 締結、リージョン East-Japan を選択 |
| OpenAI Platform (公式 API) | `data sharing for training` のオプトアウト可能 | Settings → Data Controls → OFF |
| Anthropic Claude API | デフォルト**使用しない** (公式 ToS) | Trust Center 確認、Zero Data Retention 契約 |
| Google Gemini API | プランにより異なる | Vertex AI 経由で「データ非使用」設定 |
| Snowflake Cortex | 顧客データ非使用 (公式) | Snowflake Trust Center 確認 |

#### 契約根拠の文書化テンプレ

```yaml
llm_vendor: Azure OpenAI Service
contract_type: Microsoft Data Privacy Addendum (DPA)
dpa_signed_date: 2026-01-15
opt_out_status:
  training: opted_out  # デフォルトで非使用
  abuse_monitoring: enabled  # ただし 30 日保持
  abuse_monitoring_opt_out: requested  # 申請済 (高機密用途)
region: japaneast
data_residency: 日本国内 (East Japan)
cross_border_transfer: なし
prohibited_data:
  - 要配慮個人情報
  - マイナンバー
  - クレジットカード番号
audit_trail: ServiceNow_CR_2026_001234
review_cycle: 年次
next_review: 2027-01-15
```

### ステップ 5: 海外移転対応

LLM API のリージョンが海外 (US 等) の場合、**個人情報保護法第 28 条**の適合性が必要。

```yaml
cross_border_transfer:
  destination: US (us-east-1)
  legal_basis: 本人同意 (利用規約 第 X 条 にて取得)
  alternative_basis:
    - 適合性認定 (例: EU GDPR 同等性認定)
    - 標準契約条項 (SCC) 締結
  data_minimization: PII マスキング後のみ送信
  retention: 30 日 (Abuse monitoring) → 削除確認
  user_disclosure:
    privacy_policy_url: https://example.com/privacy
    section: 「外国にある第三者への提供」セクション 4
```

### ステップ 6: 監査ログと運用

```yaml
pii_audit_log:
  - timestamp: ISO8601
  - request_id: uuid
  - user_id: hashed
  - llm_provider: enum [azure_openai, snowflake_cortex, ...]
  - region: string
  - pii_types_detected: [PHONE, EMAIL, ...]
  - masking_applied: bool
  - masking_strategy: enum [redact, tokenize, generalize]
  - pii_in_output_detected: bool  # LLM が誤って漏らしていないか
  - retention_days: 1825  # 5 年保持

monitoring:
  - metric: pii_detection_rate (1h)
    alert_threshold: 0.1  # 10% 超は異常
  - metric: pii_in_output_count (1h)
    alert_threshold: 0  # 1 件でもアラート
  - metric: opt_out_compliance_check
    schedule: 月次
    failure_action: 即時 LLM 呼び出し停止
```

## 例示: UC-003 ボイスボット (コールセンター)

### 課題

顧客が電話で「私は田中太郎、住所は東京都渋谷区...、カードは 1234-5678-9012-3456」と発話。STT で文字起こしされ LLM に送信されると PII が外部送信される。

### 防御フロー

```
[STT 出力] "私は田中太郎、住所は東京都渋谷区、カードは 1234-5678-9012-3456"
    ↓
[DLP 検出]
  - 田中太郎 → PERSON (NER)
  - 東京都渋谷区 → LOCATION (NER)
  - 1234-5678-9012-3456 → CREDIT_CARD (regex + Luhn)
    ↓
[BLOCK 判定] CREDIT_CARD は critical → LLM へ送らずエラー応答
[応答] "決済情報はお電話では承れません。マイページの URL をお送りします"
    ↓
[マスキング] "私は [PERSON_001]、住所は [LOCATION_001]、カードは [REDACTED]"
    ↓
[LLM 送信] (Azure OpenAI East-Japan、学習オプトアウト済)
    ↓
[LLM 出力] "[PERSON_001] 様、[LOCATION_001] にお住まいの件で承りました"
    ↓
[出力 DLP] PII 残存なし → OK
    ↓
[トークン展開] "田中太郎様、東京都渋谷区にお住まいの件で承りました"
    ↓
[音声合成] 顧客へ返答
```

### 録音保管時の追加防御

通話録音は 5 年保持要件があるが、**録音そのものに PII が残る**。

```yaml
recording_storage:
  primary: PII redacted text (LLM 入力前のマスク済データ)
  secondary: 暗号化された原音 (Customer-Managed Key)
  access:
    text: data_steward, compliance
    audio: compliance_only (緊急調査時のみ、二重承認)
  retention:
    text: 1825 日 (5 年)
    audio: 365 日 (1 年) → アーカイブ後 4 年は復元のみ可
```

## 出力フォーマット

| 成果物 | 形式 | 場所 |
|---|---|---|
| DLP 検出ルール | YAML | `dlp/rules/*.yaml` |
| マスキング設定 | Python module | `src/dlp/masker.py` |
| ベンダー契約根拠 | YAML | `compliance/llm-vendor-contracts.yaml` |
| 監査ログスキーマ | JSON Schema | `schemas/pii-audit.json` |
| 運用ダッシュボード | Grafana / Power BI | `dashboards/pii.json` |

## 検証方法

- [ ] PII 種別ごとの送信可否表が文書化されているか
- [ ] regex + NER (GiNZA / Presidio) が**両方**実装されているか
- [ ] Azure OpenAI / 利用 LLM の学習オプトアウト契約が締結済か (DPA/Trust Center 文書)
- [ ] リージョンが日本国内 (japaneast 等) になっているか
- [ ] 海外移転がある場合、本人同意経路が文書化されているか
- [ ] LLM 出力の PII 残存検査が実装されているか
- [ ] PII 検知率 / 出力残存率がダッシュボード化されているか
- [ ] 通話録音・チャットログの暗号化と保持期間が定義されているか

## pm-blueprint 連携

| 連携先 | 関係 |
|---|---|
| `custom/AI駆動開発リスク.md` R-AI-12 | プロンプトインジェクションが PII 漏洩経路となるため本書と連携 |
| `layer-5-risk/脅威モデリング.md` PII取扱経路 | 本書はその LLM 特化深掘り版 |
| `layer-5-risk/脅威モデリング.md` STRIDE-I (情報漏洩) | 本書の DLP / マスキングが I の主緩和策 |
| `layer-8-llm-governance/エージェント権限境界書.md` | PII カラムへのアクセス権を本書と整合 |
| `layer-8-llm-governance/LLM出力検証_許可リスト.md` | 出力 DLP との連動 |
| `layer-4-requirements/NFR.md` プライバシー NFR | 個人情報保護法準拠 NFR と本書が整合 |

## 参考

- OWASP, "OWASP Top 10 for LLM Applications 2025" LLM02
- 個人情報保護委員会「個人情報の保護に関する法律ガイドライン (通則編)」
- 個人情報保護委員会「外国にある第三者への個人データ提供」(法第 28 条解説)
- Microsoft, "Azure OpenAI Service Data Privacy Addendum"
- OpenAI, "API Data Usage Policies" (https://openai.com/policies/api-data-usage-policies)
- Anthropic, "Trust Center - Zero Data Retention"
- Microsoft Presidio, "Data Anonymization Library"
- spaCy + GiNZA Documentation (Recruit, Megagon Labs)
- PCI DSS v4.0 (Payment Card Industry Data Security Standard)
- ISO/IEC 27701 (Privacy Information Management)
