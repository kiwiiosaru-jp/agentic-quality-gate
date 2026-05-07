# トランザクション境界宣言書

> **目的:** ユースケース別にトランザクション境界・分離レベル・補償処理 (Saga/Outbox)・タイムアウトを宣言し、整合性とパフォーマンスのトレードオフを明示する。
>
> **対応する品質評価ID:** DATA-TX-001 (Critical Fail) — トランザクション境界宣言文書の不在
>
> **参照 pm-blueprint レイヤー:** Layer 3 (システム設計) / Layer 4 (NFR: Consistency/Performance)

## 基本情報

| 項目 | 内容 |
|------|------|
| プロジェクト名 | [埋めてください] |
| 対象システム | [埋めてください] |
| 作成日 | [YYYY-MM-DD] |
| 版 | 1.0 |
| 文書責任者 | [アーキテクトリード] |
| 関連 ERD | `ERD仕様書.md` |
| 関連 OpenAPI | `OpenAPI仕様書.yaml` |

---

## 1. 凡例

### 1.1 分離レベル (SQL ANSI / RDB)

| レベル | 防止する現象 | 用途例 |
|-------|-----------|-------|
| READ UNCOMMITTED | (なし) | 通常使用しない |
| READ COMMITTED | ダーティリード | 多くの一般的トランザクション |
| REPEATABLE READ | ノンリピータブルリード | 集計処理・帳票 |
| SERIALIZABLE | ファントムリード | 在庫引当・残高更新等の強整合 |
| SNAPSHOT | (RC + ノンブロッキング読み) | Azure SQL/PostgreSQL 推奨 |

### 1.2 整合性パターン

- **ローカルTx:** 単一 DB 内の ACID Tx
- **2PC:** 分散トランザクション (避けるのが原則)
- **Saga (Choreography):** 各サービスがイベントで連携、補償アクションあり
- **Saga (Orchestration):** 中央オーケストレーターが指揮
- **Outbox Pattern:** DB Tx と外部送信を Outbox テーブル経由で結合
- **Inbox Pattern:** 受信側で重複排除

---

## 2. ユースケース別 Tx 境界

### 2.1 UC-001 顧客通話録音保存 (コールセンター)

| 項目 | 内容 |
|------|------|
| 業務概要 | 通話終了時に録音メタ + STT 書き起こし + AI 要約を生成し保存 |
| 関連エンティティ | CONTACT_HISTORY, CALL_RECORDING, TRANSCRIPT, AI_SUMMARY |
| 整合性要件 | **結果整合 (eventual consistency) で十分** |
| 分離レベル | READ COMMITTED (ローカル Tx 内) |
| Tx パターン | Outbox + 非同期ジョブ (Sagaなし) |
| タイムアウト | DB Tx: 5秒 / STT: 60秒 / AI 要約: 30秒 |
| 補償処理 | 失敗時はリトライ (指数バックオフ) → 3回失敗で DLQ + 運用通知 |
| 冪等性キー | recording_id (Outbox) / contact_id (Inbox) |

**フロー:**

```
[応対終了]
   ↓
[1. CONTACT_HISTORY 更新 + Outbox INSERT (1Tx, RC)]
   ↓ (Outbox を非同期ワーカーが拾う)
[2. CALL_RECORDING 保存 (Blob)]
   ↓
[3. TRANSCRIPT 生成 (STT) → 保存]
   ↓
[4. AI_SUMMARY 生成 (LLM) → 保存]
   ↓
[Outbox 完了マーク]
```

### 2.2 UC-005 エスカレチケット起票 (コールセンター)

| 項目 | 内容 |
|------|------|
| 業務概要 | オペレーター判断でチケット起票し、担当部門に連携 |
| 関連エンティティ | TICKET, CONTACT_HISTORY, KNOWLEDGE_BASE |
| 整合性要件 | **強整合 (チケット番号重複不可)** |
| 分離レベル | SERIALIZABLE (ローカル Tx 内) |
| Tx パターン | ローカル Tx + Outbox (外部部門への通知) |
| タイムアウト | DB Tx: 3秒 |
| 補償処理 | 通知失敗時は Outbox リトライ。チケット側はロールバック不要 (起票済) |
| 冪等性キー | Idempotency-Key ヘッダ (HTTPリクエスト) |

### 2.3 UC-003 ETL データ取込 (データ分析基盤)

| 項目 | 内容 |
|------|------|
| 業務概要 | POS/EC データを Bronze→Silver→Gold に取込 |
| 関連エンティティ | BRONZE_*, SILVER_*, GOLD_* |
| 整合性要件 | **結果整合 (バッチウィンドウ内で確定)** |
| 分離レベル | SNAPSHOT (Delta Lake はバージョン管理) |
| Tx パターン | Idempotent Batch + Watermark |
| タイムアウト | バッチ全体: 4時間 / 単一ジョブ: 30分 |
| 補償処理 | Delta Lake Time Travel で巻き戻し可。前バージョンに RESTORE |
| 冪等性キー | (transaction_id, ingestion_date) 複合キー |

### 2.4 UC-010 配送配車計画作成 (物流業)

| 項目 | 内容 |
|------|------|
| 業務概要 | 朝の配送便に荷物を割当て、ドライバーを配車 |
| 関連エンティティ | SHIPMENT, DELIVERY_RUN, DRIVER |
| 整合性要件 | **強整合 (1荷物 1ドライバー)** |
| 分離レベル | SERIALIZABLE (DRIVER 在庫検証) |
| Tx パターン | ローカル Tx (単一 DB) |
| タイムアウト | DB Tx: 10秒 / 配車計画全体: 5分 |
| 補償処理 | 配車失敗時は SHIPMENT 状態を「未配車」に戻す + 運行管理者通知 |
| 冪等性キー | (planning_date, route_id) |

### 2.5 UC-014 全社 VoC 連携 API

| 項目 | 内容 |
|------|------|
| 業務概要 | 他システムから VoC 分析依頼を受け、結果を非同期返却 |
| 関連エンティティ | VOC_REQUEST, VOC_RESULT |
| 整合性要件 | **結果整合 + 重複排除** |
| 分離レベル | READ COMMITTED |
| Tx パターン | Inbox (重複排除) + ジョブキュー |
| タイムアウト | API 同期受付: 3秒 / 解析: 30分 |
| 補償処理 | 解析失敗時は status=FAILED + reason 設定して終了 (リクエスタ再投入判断) |
| 冪等性キー | Idempotency-Key ヘッダ (リクエスタ提供) |

### 2.6 UC-020 人事就業バッチ連携

| 項目 | 内容 |
|------|------|
| 業務概要 | SAP SuccessFactors から月次就業データを取込 |
| 関連エンティティ | OPERATOR_ATTENDANCE |
| 整合性要件 | **強整合 (1人月1レコード)** |
| 分離レベル | REPEATABLE READ |
| Tx パターン | ローカル Tx + Outbox (Power BI 連携) |
| タイムアウト | バッチ: 30分 |
| 補償処理 | 失敗時は処理対象月のロールバック (transaction-scoped DELETE+INSERT) |
| 冪等性キー | (year_month, employee_code) |

---

## 3. 分散トランザクション禁止リスト (避けるべきパターン)

| パターン | 禁止理由 | 代替案 |
|---------|--------|-------|
| 複数マイクロサービスにまたがる 2PC | パフォーマンス劣化、ロック保持長 | Saga / Outbox |
| Cross-region 同期 Tx | レイテンシ大、可用性低下 | リージョン内 Tx + 非同期レプリ |
| LLM 呼出を Tx 境界内に含む | LLM レイテンシ予測不可 | Tx 後に非同期呼出 |
| 外部 API 同期呼出を Tx 境界内に含む | 外部障害でロック長期化 | Tx 後に非同期呼出 (Outbox) |

---

## 4. Saga パターン使用ケース

### 4.1 Choreography Saga: 配送ルート最適化フロー

```
[依頼受付] → SHIPMENT_CREATED イベント
   ↓
[ルート計算サービス] → ROUTE_PLANNED イベント
   ↓
[配車サービス] → DELIVERY_ASSIGNED イベント
   ↓
[ドライバー通知] → NOTIFICATION_SENT
```

**補償アクション:**

| 失敗ステップ | 補償 |
|-----------|------|
| 配車失敗 | SHIPMENT を「未配車」へ + 運行管理者通知 |
| 通知失敗 | 配車取消 + 再配車キュー投入 |

### 4.2 Orchestration Saga: 大規模 LLM ワークフロー

```
[Orchestrator] ──▶ STT サービス
              ──▶ LLM 要約サービス
              ──▶ センチメント分析
              ──▶ 統合・保存
```

**補償:** 各ステップ失敗時はサービス別の補償エンドポイントを Orchestrator が呼出

---

## 5. 同時実行制御 / ロック方針

| シナリオ | 採用方式 | 理由 |
|---------|---------|------|
| 在庫引当 | 楽観ロック (バージョン番号) | コンフリクト稀。リトライで対応可 |
| 残高更新 | 悲観ロック (SELECT FOR UPDATE) | コンフリクト頻繁。失敗で業務停止 |
| 配車計画 | アドバイザリーロック (Postgres pg_advisory_lock) | 計画粒度のロック保護 |
| マスタ更新 | バッチ排他 (アプリレベル mutex) | 単発バッチが同時起動しない |

---

## 6. 性能要件と Tx 設計の関係

| UC | TPS | DBラウンドトリップ | Tx 時間目標 | 設計影響 |
|----|-----|---------------|-----------|---------|
| UC-001 | [10] | [3 回] | <100ms | RC + Outbox |
| UC-005 | [1] | [4 回] | <500ms | SERIALIZABLE OK |
| UC-003 | [batch] | (バッチ) | <30分/job | Snapshot |
| UC-010 | [0.1] | [10 回] | <10s | SERIALIZABLE |
| UC-014 | [3] | [2 回] | <100ms (受付) | RC + Inbox |

---

## 7. 死活監視とアラート

| メトリクス | 閾値 | アクション |
|----------|------|---------|
| Outbox 滞留件数 | > 1000 | アラート (P2) |
| Outbox 失敗率 | > 5% | アラート (P2) |
| デッドロック発生率 | > 0.1% | アラート (P3) |
| Long-running Tx | > 30秒 | アラート (P2) + 自動 KILL |
| Saga 補償発生率 | > 1% | アラート (P3) |

---

## 8. 開発・運用ガイドライン

### 8.1 開発者向けチェックリスト

- [ ] Tx 境界内に外部 API・LLM 呼出を含めていない
- [ ] 必要最小の分離レベルを選択している
- [ ] タイムアウトを明示している
- [ ] 冪等性キーを定義している
- [ ] 補償処理を設計している (Saga 利用時)

### 8.2 運用向けチェックリスト

- [ ] Outbox/Inbox テーブルの監視ダッシュボードあり
- [ ] Saga 状態の追跡ダッシュボードあり
- [ ] 補償アクションの実行ログ保管
- [ ] DLQ メッセージの定期レビュー (週次)

---

## 9. 変更履歴

| 日付 | 版 | 変更内容 | 変更者 |
|------|----|---------|-------|
| YYYY-MM-DD | 1.0 | 初版作成 | [...] |

---

## 10. pm-blueprint 連携

- **出力レイヤー:** Layer 3 (System Design) / Layer 4 (NFR: Consistency)
- **関連品質ゲート ID:**
  - DATA-TX-001 (Critical) ... 本宣言書の存在
  - DATA-ENTITY-001 (Critical) ... ERD仕様書との整合
  - API-IDEMPOTENCY-001 (Critical) ... OpenAPI 仕様書 (Idempotency-Key) との整合
- **ゲート通過条件:**
  1. 全主要 UC についてTx パターン・分離レベル・タイムアウトが宣言済
  2. Saga/Outbox 採用箇所の補償アクションが定義済
  3. 死活監視のメトリクスとアラート閾値が設定済
- **連携 ADR:** Tx パターン重要決定は ADR 化 (例: ADR-XXXX「LLM 呼出は Tx 外に置く」)
