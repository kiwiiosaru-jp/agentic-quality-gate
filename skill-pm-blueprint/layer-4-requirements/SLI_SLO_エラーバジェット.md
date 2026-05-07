---
name: SLI_SLO_エラーバジェット
description: SRE 流の SLI 定義 / SLO 目標 / エラーバジェット / バーンダウン速度 / リリース凍結基準 / 計測ダッシュボードを設計するスキル。品質ゲート NFR-SLO-001 に対応。
---

# SLI / SLO / エラーバジェット (Service Level Indicators / Objectives / Error Budget)

## 対応する品質ゲート

| ID | 観点 | 既定の Conditional 条件 |
|----|------|------|
| NFR-SLO-001 | SLO とエラーバジェット | SLI/SLO の定義、エラーバジェット計算、バーン速度監視、リリース凍結基準が宣言されていない |

## 概要

「99.9% の可用性」という宣言は、それだけでは運用の指標にならない。SRE 流に分解すると:

- **SLI (Indicator)** : 何のメトリクスで計測するか (例: 5xx 率)
- **SLO (Objective)** : どの値を目指すか (例: 99.9% over 30d)
- **エラーバジェット** : SLO の残り (例: 0.1% = 月 43.2 分)
- **バーン速度** : エラーバジェットを消費するスピード
- **リリース凍結基準** : バーン速度が異常な時に新規デプロイを止める仕組み

これらを **数値・式・トリガ** として定義しておくと、運用判断が個人の勘ではなく組織の合意になる。

## いつ使うか

- 本番リリース前 (運用 SLA を IT 部門・業務部門で握る時)
- 既存サービスの可用性が論争になった時 (誰かが「最近遅い」と言い、誰かが「いや問題ない」と言う)
- リリース速度を上げたい / 安全装置を入れたい時
- 上司や顧客に「サービスは安定しているか」を数値で答える必要がある時

## 手順 (5 ステップ)

### ① SLI を選ぶ (測定可能な指標)

SLI は **「ユーザー視点で良い状態か悪い状態か」** を判定できる指標を選ぶ。RED method / USE method / Four Golden Signals を参考に。

#### 推奨 SLI カテゴリ (Google SRE 流)

| カテゴリ | 例 |
|---------|-----|
| **Availability** | 成功率 = (全リクエスト数 - エラー数) / 全リクエスト数 |
| **Latency** | リクエストの 95/99 パーセンタイル応答時間 |
| **Throughput** | 単位時間あたり処理件数 |
| **Quality** | 正しい結果を返した率 (LLM 応答の品質、推論精度) |
| **Freshness** | データの最新性 (Bronze→Gold の遅延) |
| **Correctness** | 集計結果の正答率 |

#### SLI 定義テンプレート

```yaml
slis:
  - id: SLI-API-001
    name: API 成功率
    formula: (count(status_code < 500) / count(status_code != null)) over 5min window
    metric_source: Azure Application Insights
    valid_event_filter: status_code IS NOT NULL AND user_agent NOT LIKE '%bot%'
    excluded_endpoints: [/health, /metrics]
  - id: SLI-API-002
    name: API レイテンシ P95
    formula: percentile(duration_ms, 95) over 5min window
    metric_source: Azure Application Insights
    excluded_endpoints: [/health, /metrics, /admin/*]
  - id: SLI-DATA-001
    name: Gold 層データ鮮度
    formula: max(now() - max(processed_at)) per partition
    metric_source: Databricks Audit Log
    target_partition: silver_interaction_analysis
```

### ② SLO を決める (目標値とウィンドウ)

SLI に対する**目標値** と**評価ウィンドウ**を決める。

#### 推奨ウィンドウ

- 短期: 5min / 1h (バーン速度監視)
- 中期: 7d / 30d (主たる SLO)
- 長期: 90d (四半期レビュー)

#### SLO の階層

| 階層 | 値 | 意味 |
|-----|-----|------|
| **撤退 (Floor)** | 99.0% | これ以下はサービス停止級 |
| **約束 (SLO)** | 99.9% | 顧客と握る公式 SLO |
| **目標 (Stretch)** | 99.95% | 内部目標。これを超えると過剰投資 |

#### 99.9% / 99.95% / 99.99% の意味

| SLO | 月間ダウンタイム上限 | 年間ダウンタイム上限 |
|-----|-----------|-----------|
| 99.0% | 約 7.3h | 約 87.6h |
| 99.5% | 約 3.65h | 約 43.8h |
| 99.9% | 約 43.2 分 | 約 8.76h |
| 99.95% | 約 21.6 分 | 約 4.38h |
| 99.99% | 約 4.38 分 | 約 52.6 分 |

#### SLO 宣言テンプレート

```yaml
slos:
  - id: SLO-API-001
    sli: SLI-API-001 (API 成功率)
    objective: 99.9% over rolling 30 days
    error_budget_per_30d: 0.1% (≒ 43.2 分)
    burn_rate_alerts:
      page: burn rate > 14.4× sustained 1h  (2% budget in 1h)
      ticket: burn rate > 6× sustained 6h    (10% budget in 6h)
    consequences_when_breached:
      - リリース凍結
      - インシデントレビュー実施
      - 翌四半期に予算配分見直し
  - id: SLO-API-002
    sli: SLI-API-002 (API P95)
    objective: P95 < 1000ms over rolling 7 days
    burn_rate_alerts:
      page: P95 > 2000ms sustained 15 min
```

### ③ エラーバジェットを計算する (NFR-SLO-001)

**エラーバジェット = (1 - SLO) × ウィンドウ内総イベント数**

#### 計算例

```
SLO: 99.9% (= 0.001 のエラー許容)
30 日間の総リクエスト数: 30,000,000
エラーバジェット = 0.001 × 30,000,000 = 30,000 リクエスト
                = 月間 43.2 分のダウンタイム相当
                = 100万件あたり 1,000 件のエラー (= 1000ppm)
```

**「29ppm」(参照仕様)の解釈:**

```
99.9971% SLO → 0.0029% エラー許容 = 29ppm (parts per million)
30,000,000 req/30d で エラー上限 870 件
ダウンタイム上限: 月 12.5 分
99.99% (4.38 分) と 99.999% (26 秒) の中間
```

#### バーンダウン (Burn Rate) の指標化

**Burn Rate** = エラーバジェット消費速度。1.0 が「ちょうど予算通り」。

| Burn Rate | 意味 | 推奨アラート |
|-----------|------|----------|
| 1.0× | 予算通りに 30 日で消費 | なし |
| 2.0× | 15 日で消費 | 注意 (Slack) |
| 6.0× | 5 日で消費 | チケット (Jira) |
| 14.4× | 約 50 時間で消費 | **ページャ (PagerDuty)** |
| 36× | 約 20 時間で消費 | **即応 + リリース凍結** |

#### Multi-window / Multi-burn-rate アラート (Google SRE 流)

```yaml
alerts:
  - severity: page
    condition: |
      (burn_rate(1h) > 14.4 AND burn_rate(5m) > 14.4) OR
      (burn_rate(6h) > 6   AND burn_rate(30m) > 6)
    action: PagerDuty 即応
  - severity: ticket
    condition: |
      (burn_rate(24h) > 3 AND burn_rate(2h) > 3) OR
      (burn_rate(72h) > 1 AND burn_rate(6h) > 1)
    action: Jira Issue 起票, 翌営業日対応
```

### ④ リリース凍結基準を決める

エラーバジェットが枯渇 (or 急速消費) したら**新規デプロイを止める** ルール。

#### 推奨ルール

```yaml
release_freeze:
  trigger:
    - burn_rate(1h) > 14.4  (2% を 1 時間で消費)
    - error_budget_remaining_30d < 25%  (75% 以上消費)
  duration:
    - 最低 24h or バーン率が 1.0 以下に戻るまで
  exceptions:
    - 緊急セキュリティパッチ (CISO 承認)
    - エラーを修正するデプロイ (PO 承認)
  unlock_criteria:
    - エラー要因の RCA 完了
    - 再発防止策が PR で merge
    - SLO ダッシュボードで回復確認
```

#### コミュニケーション

```yaml
freeze_comms:
  notify_channels: [#deploy-announce, #incident-room, all-eng@]
  message_template: |
    リリース凍結発動 (SLO-API-001 burn rate 18.2)
    開始: 2026-04-30 14:30 JST
    解除予定: 翌日 RCA 完了後
    例外申請: #release-exception
```

### ⑤ ダッシュボードで可視化する

Grafana / Azure Workbook / Datadog 等で、以下を 1 画面に集約。

#### 必須ウィジェット

1. **SLI 現在値** (大きく数字で)
2. **SLO バーン率** (1h / 6h / 24h)
3. **エラーバジェット残量** (棒グラフ % 残)
4. **バーンダウン推移** (30 日トレンド)
5. **トップエラー** (status × endpoint)
6. **直近インシデント** (PagerDuty 連携)

#### 例: Azure Monitor Workbook KQL

```kql
// 過去 30 日間の SLO 達成率
let window = 30d;
let total = requests
  | where timestamp > ago(window)
  | where url !contains "/health"
  | count;
let errors = requests
  | where timestamp > ago(window)
  | where resultCode >= 500
  | where url !contains "/health"
  | count;
print
  total = toscalar(total),
  errors = toscalar(errors),
  success_rate = 1.0 - (toscalar(errors) * 1.0 / toscalar(total)),
  budget_remaining = (1.0 - toscalar(errors) * 1.0 / toscalar(total) - 0.999) / 0.001
```

```kql
// バーン率 (直近 1h vs 30 日 SLO)
let total_1h = requests | where timestamp > ago(1h) | count;
let errors_1h = requests | where timestamp > ago(1h) | where resultCode >= 500 | count;
print burn_rate_1h = (toscalar(errors_1h) * 1.0 / toscalar(total_1h)) / 0.001
```

## 出力フォーマット

```
docs/
├── reliability/
│   ├── 01_slis.yaml              # SLI 定義
│   ├── 02_slos.yaml              # SLO 目標
│   ├── 03_error_budget_policy.md # バジェット枠 + 凍結ポリシー
│   └── 04_dashboards.md          # ダッシュボード仕様
├── adr/
│   └── 0040-slo-policy.md
.github/workflows/
└── release-gate.yml              # SLO バーン率を CI でチェック
```

## 例示: コールセンター / VoC 基盤の SLI/SLO 一覧

| SLI | SLO | 30d Budget | アラート |
|-----|-----|-----------|--------|
| API 成功率 (5xx 除外) | 99.95% over 30d | 21.6 分 | burn 14.4× で page |
| API レイテンシ P95 | < 800ms over 7d | - | P95>1500ms で page |
| Genesys Webhook 受信成功率 | 99.99% over 30d | 4.38 分 | burn 14.4× で page |
| Bronze→Gold 鮮度 | 95% < 15 min over 30d | 5% over 15min | 30 件超過で ticket |
| LLM 分類精度 | 90% over 7d | - | 精度低下で ticket |
| Azure OpenAI スロットリング | < 0.1% over 7d | 0.1% req | rate 増で ticket |

## 数値 SLO の根拠 (実例)

```yaml
slo_rationale:
  api_success_rate:
    chosen: 99.95%
    rationale: |
      コールセンターの問合せ受付は 24/365 業務だが、
      99.99% は冗長コスト過大、99.9% (43.2 分) は業務影響大。
      99.95% (21.6 分) を採用、Active-Passive で達成見込。
    business_impact_if_breached:
      - 1 分の停止 = 約 5 件の問合せ取りこぼし
      - 21 分超過 = 業務部門エスカレ、四半期 KPI 影響
  data_freshness:
    chosen: 95% < 15 min
    rationale: |
      コールセンタースーパーバイザの「リアルタイム分析」は
      実は 15 分遅延で十分 (シフト交代単位で確認)。
      99% を狙うと Spark Structured Streaming 必須でコスト 3x。
```

## 作成時のチェックリスト

- [ ] 主要なユーザー操作ごとに SLI が定義されている
- [ ] SLI は「成功率」「レイテンシ」「鮮度」のいずれかで明確に計算式がある
- [ ] SLO 値・ウィンドウ (7d / 30d) が宣言されている
- [ ] エラーバジェット (時間 / 件数 / ppm) が計算されている
- [ ] Multi-window / Multi-burn-rate のアラートが定義されている
- [ ] Burn rate 14.4× / 6× の page / ticket 区分けがある
- [ ] リリース凍結基準が文書化されている
- [ ] 凍結時の例外承認フローが定義されている
- [ ] ダッシュボード (Workbook / Grafana / Datadog) のクエリが提供されている
- [ ] 顧客向け SLA と内部 SLO の差分が明記されている
- [ ] 四半期 SLO レビュー会の開催が予定されている
- [ ] ADR (SLO ポリシー) が起票済み

## pm-blueprint 連携

| 既存サブスキル | 連携ポイント |
|--------------|-----------|
| `layer-4-requirements/SMART非機能要件.md` | NFR の Reliability/Performance ランディングゾーンを SLI/SLO 化 |
| `layer-4-requirements/可観測性三本柱.md` | SLI 計測の Metric / Trace / Log を統合 |
| `layer-4-requirements/RTO_RPO_DR設計.md` | Floor (99.0%) を下回るとき RTO/RPO 発動 |
| `layer-3-architecture/API設計.md` | API SLO は OpenAPI changelog と連動 |
| `layer-3-architecture/データ設計詳細.md` | データ鮮度 SLO は Eventually Consistent 遅延 SLA と一致 |

## 品質ゲート対応サマリ

| ゲート ID | 本スキル該当節 | 出力アーティファクト |
|----------|--------------|------------------|
| NFR-SLO-001 | 手順 ①〜⑤ | `docs/reliability/{01..04}` + ADR-0040 |

## 参考

- Google SRE Book "Implementing SLOs" (Chapter 4): https://sre.google/sre-book/service-level-objectives/
- Google SRE Workbook "Alerting on SLOs" (Multi-window / Multi-burn-rate)
- AWS Builders' Library "Implementing health checks"
- Microsoft Learn "Reliability principles" (Well-Architected Framework)
- Datadog "SLO guide" / Grafana "SLO dashboards"
- Charity Majors, "Observability Engineering"
- Niall Murphy et al, "Implementing Service Level Objectives" (O'Reilly 2020)
