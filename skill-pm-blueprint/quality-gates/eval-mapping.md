# 品質ゲート ↔ pm-blueprint Layer 逆引きマップ

> agentic-quality-gate v4 の 49 評価エントリと pm-blueprint レイヤー出力ファイルの対応表
> 用途: 品質ゲート Fail/Conditional の指摘 → 対応する pm-blueprint レイヤー特定

## 1. Layer → 品質ゲート ID 一覧 (順引き)

### Layer 1 経営層
- STRATEGY-LIFETIME-001 (寿命宣言)
- COST-ESTIMATE-001 (概算コスト)
- COST-UNIT-001 (課金単位)

### Layer 2 仮説 (前提検証材料、直接対応なし)
- (前提検証段階のため評価対象外、矢羽①完了後の Step 6 経営判断で確認)

### Layer 3 アーキテクチャ
- **基本 (既存)**:
  - ARCH-DDD-001 (DDDコンテキストマップ.md, ADR作成.md)
  - ARCH-LAYER-001 (ADR作成.md)
- **データ設計詳細 (新規)**:
  - DATA-ENTITY-001 (データ設計詳細.md)
  - DATA-TX-001
  - DATA-OUTBOX-001
  - DATA-DELIVERY-001
  - DATA-CONSISTENCY-001
  - DATA-HISTORY-001
  - DATA-SOT-001
- **API設計 (新規)**:
  - API-IDEMPOTENCY-001 (API設計.md)
  - API-STYLE-001
  - API-VERSION-001
- **アーキスタイル (新規, Phase 2)**:
  - ARCH-MONOLITH-001 (アーキスタイル選定.md)
  - ARCH-STYLE-001
- **環境・IaC (新規, Phase 2)**:
  - ENV-SEPARATE-001 (環境分離_IaC.md)
  - ENV-DATAFLOW-001

### Layer 4 要件
- **基本 (既存)**:
  - (UC/NFR 各エントリは内部参照)
- **NFR運用 (新規)**:
  - NFR-SLO-001 (SLI_SLO_エラーバジェット.md)
  - NFR-RTORPO-001 (RTO_RPO_DR設計.md)
  - NFR-OBS-001 (可観測性三本柱.md)
- (PERF-SCALE-001 / PERF-CACHE-001 / PERF-STATELESS-001 は Layer 4 で部分対応)

### Layer 5 リスク・セキュリティ
- **基本 (既存)**:
  - LEGAL-THREATMODEL-001 (脅威モデリング.md = STRIDE)
- **セキュリティ詳細 (新規, Phase 2)**:
  - SEC-AUTHZ-DESIGN-001 (認可方式選定.md)
  - SEC-CRYPTO-001 (暗号化_KMS設計.md)
  - SEC-ZEROTRUST-001 (ゼロトラスト方針.md)
  - GOV-INTERNAL-SYSTEM-001 (ゼロトラスト方針.md と同一)
  - LEGAL-THREATMODEL-001 (LINDDUNプライバシー脅威.md ※STRIDE補完)

### Layer 6 実行 (WBS)
- (各サブスキルが矢羽⑥ 非機能・運用への組込で運用領域へ)

### Layer 7 法務・コンプライアンス (新規)
- LEGAL-PII-001 (個人情報取扱台帳.md)
- COMPLY-PIPA-JP-001 (個情法対応マトリクス.md)
- LEGAL-INDUSTRY-001 (業法該当性判定書.md)
- LEGAL-DATACLASS-001 (データ分類台帳.md)
- LEGAL-AIGEN-001 (AI生成物著作権ポリシー.md)
- LEGAL-CROSSBORDER-001 (越境データ移転書.md)
- LEGAL-LICENSE-001 (OSSライセンス管理.md)
- COMPLY-RETAIN-001 (データ保持_削除戦略.md)
- DATA-DELETE-001 (データ保持_削除戦略.md)
- COMPLY-GDPR-001 (該当時 N/A、海外案件で発動)

### Layer 8 LLM ガバナンス (新規)
- LLM-AGENCY-001 (エージェント権限境界書.md)
- LLM-INJECT-DIR-001 (プロンプトインジェクション評価.md)
- LLM-INJECT-INDIR-001 (プロンプトインジェクション評価.md)
- LLM-OUTPUT-TRUST-001 (LLM出力検証_許可リスト.md)
- LLM-PII-001 (PII境界_DLP.md)
- LLM-HALLUCIN-001 (ハルシネーション対策.md)
- LLM-DRIFT-001 (モデルドリフト検知.md)
- LLM-EVAL-REGR-001 (評価セット_回帰検知.md)
- GOV-AI-DATAFLOW-001 (AIデータ境界ガイド.md ※Layer 9 連携)
- LLM-COST-RUNAWAY-001 (Layer 5 リスクレジスタ参照)
- LLM-TOOL-POISON-001 (該当時、ツール連携あり)
- LLM-LIB-HALLUCIN-001 (該当時、AI コード生成あり)

### Layer 9 運用 (新規)
- DOC-RUNBOOK-001 (ランブック規約.md)
- GOV-AUDIT-TRAIL-001 (監査証跡設計.md)
- GOV-SHADOW-AI-001 (シャドーAI禁止ガバナンス.md)
- GOV-AI-DATAFLOW-001 (AIデータ境界ガイド.md ※Layer 8 連携)

## 2. 品質ゲート ID → Layer (逆引き)

### P0 Critical
| ID | Layer | サブスキル |
|----|-------|----------|
| LEGAL-INDUSTRY-001 | 7 | 業法該当性判定書 |
| LEGAL-PII-001 | 7 | 個人情報取扱台帳 |
| LEGAL-TOS-001 | 7 | (該当時、外部データ取得時) |

### P0 High
| ID | Layer | サブスキル |
|----|-------|----------|
| COST-ESTIMATE-001 | 1, 6 | 意思決定テンプレート, WBS |
| COST-UNIT-001 | 1 | 意思決定テンプレート |
| LEGAL-AIGEN-001 | 7 | AI生成物著作権ポリシー |
| LEGAL-CROSSBORDER-001 | 7 | 越境データ移転書 |
| LEGAL-DATACLASS-001 | 7 | データ分類台帳 |
| LEGAL-LICENSE-001 | 7 | OSSライセンス管理 |
| LEGAL-THREATMODEL-001 | 5 | LINDDUNプライバシー脅威, 脅威モデリング |
| STRATEGY-LIFETIME-001 | 1 | 意思決定テンプレート |

### P1 Critical
| ID | Layer | サブスキル |
|----|-------|----------|
| API-IDEMPOTENCY-001 | 3 | API設計 |
| ARCH-LAYER-001 | 3 | アーキスタイル選定, ADR作成 |
| DATA-ENTITY-001 | 3 | データ設計詳細 |
| DATA-TX-001 | 3 | データ設計詳細 |
| ENV-SEPARATE-001 | 3 | 環境分離_IaC |
| SEC-AUTHZ-DESIGN-001 | 5 | 認可方式選定 |
| SEC-TENANT-001 | 3, 5 | (SaaS/B2Bの場合) |

### P1 High
| ID | Layer | サブスキル |
|----|-------|----------|
| API-STYLE-001 | 3 | API設計 |
| API-VERSION-001 | 3 | API設計 |
| ARCH-AGGREGATE-001 | 3 | DDDコンテキスト |
| ARCH-DDD-001 | 3 | DDDコンテキスト |
| ARCH-MONOLITH-001 | 3 | アーキスタイル選定 |
| ARCH-STYLE-001 | 3 | アーキスタイル選定 |
| DATA-CONSISTENCY-001 | 3 | データ設計詳細 |
| DATA-DELETE-001 | 3, 7 | データ設計詳細, データ保持_削除戦略 |
| DATA-DELIVERY-001 | 3 | データ設計詳細 |
| DATA-HISTORY-001 | 3 | データ設計詳細 |
| DATA-OUTBOX-001 | 3 | データ設計詳細 |
| DATA-SOT-001 | 3 | データ設計詳細 |
| ENV-DATAFLOW-001 | 3 | 環境分離_IaC |
| NFR-OBS-001 | 4 | 可観測性三本柱 |
| NFR-RTORPO-001 | 4 | RTO_RPO_DR設計 |
| NFR-SLO-001 | 4 | SLI_SLO_エラーバジェット |
| PERF-SCALE-001 | 4 | (NFR Scalability、新規スキル不要) |
| PERF-STATELESS-001 | 3 | (Web系の場合) |
| SEC-CRYPTO-001 | 5 | 暗号化_KMS設計 |
| SEC-ZEROTRUST-001 | 5 | ゼロトラスト方針 |

### Cross-cutting Critical
| ID | Layer | サブスキル |
|----|-------|----------|
| COMPLY-PIPA-JP-001 | 7 | 個情法対応マトリクス |
| COMPLY-GDPR-001 | 7 | (海外案件時) |
| GOV-AI-DATAFLOW-001 | 8, 9 | PII境界_DLP, AIデータ境界ガイド |
| GOV-SHADOW-AI-001 | 9 | シャドーAI禁止ガバナンス |
| LLM-AGENCY-001 | 8 | エージェント権限境界書 |
| LLM-INJECT-DIR-001 | 8 | プロンプトインジェクション評価 |
| LLM-INJECT-INDIR-001 | 8 | プロンプトインジェクション評価 |
| LLM-OUTPUT-TRUST-001 | 8 | LLM出力検証_許可リスト |
| LLM-PII-001 | 8 | PII境界_DLP |
| LLM-TOOL-POISON-001 | 8 | (ツール連携時) |

### Cross-cutting High
| ID | Layer | サブスキル |
|----|-------|----------|
| COMPLY-RETAIN-001 | 7 | データ保持_削除戦略 |
| DOC-RUNBOOK-001 | 9 | ランブック規約 |
| GOV-AUDIT-TRAIL-001 | 9 | 監査証跡設計 |
| GOV-INTERNAL-SYSTEM-001 | 5 | ゼロトラスト方針 |
| LLM-COST-RUNAWAY-001 | 5, 8 | リスクレジスタ R-002 + Layer 8 |
| LLM-DRIFT-001 | 8 | モデルドリフト検知 |
| LLM-EVAL-REGR-001 | 8 | 評価セット_回帰検知 |
| LLM-HALLUCIN-001 | 8 | ハルシネーション対策 |
| LLM-LIB-HALLUCIN-001 | 8 | (AIコード生成時) |

## 3. Fail 解消フロー (例)

### 例1: LEGAL-PII-001 Fail
1. eval-mapping.md でこの ID を検索 → Layer 7 個人情報取扱台帳.md が担当
2. `~/.claude/skills/pm-blueprint/layer-7-legal-compliance/個人情報取扱台帳.md` の手順に従い文書作成
3. `~/.claude/skills/pm-blueprint/templates/PII取扱台帳.yml` を雛形として埋める
4. 法務 + DPO レビューを実施 (人間アクション)
5. 計画書 セクション13 に追加
6. 再評価で Pass 確認

### 例2: API-IDEMPOTENCY-001 Fail
1. Layer 3 API設計.md 担当
2. UC-005, UC-014, UC-020 (書込API) を識別
3. OpenAPI仕様書.yaml テンプレで Idempotency-Key ヘッダ仕様化
4. ARB レビュー
5. 再評価で Pass 確認

### 例3: LLM-AGENCY-001 Fail
1. Layer 8 エージェント権限境界書.md 担当
2. UC-003 (ボイスボット), UC-011 (Text2SQL) を対象に境界書作成
3. Read-only 限定 / DDL/DML 禁止のサンドボックス設計
4. CISO + AI 倫理委員会レビュー
5. 再評価で Pass 確認

## 4. ループ判定基準

- すべての Fail 解消 (Critical=0)
- High Conditional ≤ 5 (人間レビュー待ちのみ)
- Pass 率 ≥ 80%

## 参考
- `agentic-quality-gate/knowledge/INDEX.md` 全176件ナレッジ
- `quality-gates/checklist.yml` 49項目定義
- `custom/品質ゲート連携.md` 連携フロー
