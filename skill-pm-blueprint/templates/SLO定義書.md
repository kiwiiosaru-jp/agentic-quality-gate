# SLO 定義書 (Service Level Objective Specification)

> **目的:** SLI (Service Level Indicator) を定義し、SLO 目標値、エラーバジェット、バーンレート (バーンダウン速度)、計測ダッシュボードを明示する。
>
> **対応する品質評価ID:** NFR-SLO-001 (High Conditional) — SLI/SLO/エラーバジェット/ダッシュボード未定義
>
> **参照 pm-blueprint レイヤー:** Layer 4 (NFR: Reliability/Performance) / Layer 6 (Operations/Observability)

## 基本情報

| 項目 | 内容 |
|------|------|
| プロジェクト名 | [埋めてください] |
| 対象サービス | [埋めてください 例: 顧客サービス基盤 API] |
| 計測単位 | rolling 28-day window |
| 文書責任者 | [SREリード / オペレーションリード] |
| 作成日 | [YYYY-MM-DD] |
| 版 | 1.0 |
| 次回見直し | [YYYY-MM-DD 四半期1回] |
| 関連文書 | `可観測性方針.md`, `DR設計書.md`, `ランブック規約.md` |

---

## 1. SLO 概念整理

### 1.1 SLI (Service Level Indicator)

サービス品質を定量化する **計測値** (例: 成功率、レイテンシ)。

### 1.2 SLO (Service Level Objective)

SLI に対する **目標値** (例: 「成功率 99.9%」)。

### 1.3 SLA (Service Level Agreement)

SLO に基づく **顧客との契約** (達成しない場合の補償条項あり)。

### 1.4 エラーバジェット

許容される失敗の総量。

| SLO | 月間ダウンタイム | 月間エラーバジェット (req) |
|-----|---------------|---------------------|
| 99.0% | 7.2 時間 | 10,000 req/M = 100 req失敗 |
| 99.5% | 3.6 時間 |  |
| 99.9% | 43.2 分 | 10,000 req/M = 10 req失敗 |
| 99.95% | 21.6 分 |  |
| 99.99% | 4.32 分 | 10,000 req/M = 1 req失敗 |

### 1.5 29ppm の計算例

99.9971% (29ppm = 0.00029% の失敗率) の場合:

```
1,000,000 req のうち失敗許容: 29 req
1ヶ月の API リクエスト想定: 30,000,000 req → 失敗許容 870 req/月
1日あたりエラーバジェット: 約 29 req
```

### 1.6 バーンレート (Burn Rate)

エラーバジェット消費速度 = 実エラー率 / SLO 許容エラー率。

| バーンレート | 解釈 | アラート例 |
|------------|------|---------|
| 1.0 | 計画通り消費 | (アラートなし) |
| 2.0 | 2 倍速消費 → 半月でバジェット枯渇 | warning |
| 5.0 | 5 倍速消費 → 数日で枯渇 | critical |
| 10.0 | 10 倍速消費 → 1日未満で枯渇 | page (即時通知) |

---

## 2. SLI / SLO 定義

### 2.1 サービス可用性 (Availability)

| 項目 | 定義 |
|------|------|
| **SLI 名** | api_availability_rate |
| **計測対象** | /v1/* エンドポイントへの全リクエスト |
| **数式** | (HTTP 2xx + 3xx の合計) / (全レスポンス総数) |
| **除外** | クライアント起因の 4xx (401, 403, 404, 422 等) |
| **計測ウィンドウ** | rolling 28-day |
| **SLO 目標** | **99.9%** (許容ダウンタイム 43.2 分/月) |
| **エラーバジェット (28d)** | 約 40 分相当 |
| **計測ツール** | Azure Monitor → Log Analytics → KQL |
| **ダッシュボード URL** | `[https://portal.azure.com/.../slo-availability]` |
| **担当** | [SREリード] |

### 2.2 エンドツーエンド レイテンシ (P95)

| 項目 | 定義 |
|------|------|
| **SLI 名** | api_latency_p95_ms |
| **計測対象** | /v1/tickets, /v1/voc/analyses 主要 5 エンドポイント |
| **数式** | 95 パーセンタイルのレスポンス時間 |
| **計測ウィンドウ** | rolling 28-day |
| **SLO 目標** | **P95 < 800ms** |
| **エラーバジェット** | 5% のリクエストで 800ms 超過まで許容 |
| **計測ツール** | Application Insights → KQL |
| **ダッシュボード URL** | `[https://portal.azure.com/.../slo-latency]` |
| **担当** | [APIリード] |

### 2.3 通話録音処理パイプライン (コールセンター業界)

| 項目 | 定義 |
|------|------|
| **SLI 名** | call_pipeline_e2e_minutes |
| **計測対象** | 通話終了 → AI 要約 完了までの所要時間 |
| **数式** | (AI_SUMMARY.generated_at - CONTACT_HISTORY.ended_at) の P95 |
| **計測ウィンドウ** | rolling 28-day |
| **SLO 目標** | **P95 < 10 分** |
| **計測ツール** | Databricks ジョブ + KQL |
| **ダッシュボード URL** | `[...]` |
| **担当** | [データエンジニアリード] |

### 2.4 配送遅延通知 (物流業界)

| 項目 | 定義 |
|------|------|
| **SLI 名** | shipment_alert_latency_min |
| **計測対象** | 遅延発生 → 顧客通知送信までの時間 |
| **SLO 目標** | **P99 < 30 分** |
| **計測ツール** | Event Grid + App Insights |
| **担当** | [配送システムリード] |

### 2.5 BI ダッシュボード鮮度 (データ分析基盤)

| 項目 | 定義 |
|------|------|
| **SLI 名** | bi_data_freshness_hours |
| **計測対象** | Gold 層データ最終更新時刻 |
| **SLO 目標** | **24 時間以内 99% 達成** |
| **計測ツール** | Databricks Job Status + Power BI Last Refresh |
| **担当** | [BIリード] |

### 2.6 LLM 応答品質 (AI ガバナンス)

| 項目 | 定義 |
|------|------|
| **SLI 名** | llm_response_acceptance_rate |
| **計測対象** | LLM 出力をユーザーが採用 (編集不要) した率 |
| **SLO 目標** | **> 80%** |
| **計測ツール** | フィードバック UI + ログ |
| **担当** | [AIリード] |

---

## 3. エラーバジェットポリシー

### 3.1 定義

エラーバジェット = SLO 許容失敗総量 - 実消費。

### 3.2 28日間の予算と消費アラート

| 残量 | アクション |
|------|---------|
| > 50% | 通常開発継続 |
| 25%-50% | 新機能リリース時に変更影響レビュー強化 |
| 10%-25% | リリースフリーズ検討 + 信頼性タスク優先化 |
| < 10% | リリースフリーズ + 緊急体制 + 経営層報告 |
| 0% (枯渇) | 強制フリーズ + 機能停止 / 機能縮退検討 |

### 3.3 バーンレートアラート

| バーンレート閾値 | 評価ウィンドウ | 通知レベル |
|-------------|------------|---------|
| 14.4x | 1時間 | page (緊急ページャ) |
| 6x | 6時間 | critical (Slack/PagerDuty) |
| 1x | 24時間 | warning (Slack) |

(参考: Google SRE workbook の Multi-Window Multi-Burn-Rate)

---

## 4. 計測ダッシュボード

### 4.1 ダッシュボード一覧

| ダッシュボード | URL | 目的 | 更新頻度 |
|------------|-----|------|--------|
| SLO 全体俯瞰 | `[埋めてください]` | エグゼクティブ向け月次 | 1分 |
| サービス別 SLO | `[...]` | 開発チーム向け | 1分 |
| エラーバジェット燃焼 | `[...]` | バーンレート監視 | 5分 |
| インシデント Postmortem | `[...]` | 対応振り返り | 随時 |

### 4.2 ダッシュボード必須要素

- 過去 28 日 SLO 達成率
- 現在のバーンレート (1h / 6h / 24h)
- 主要エラー Top 10
- インシデント発生回数 (重大度別)
- ヒートマップ: 時間帯×エンドポイント別エラー率

### 4.3 ダッシュボード サンプル KQL (Azure Monitor)

```kql
// API 可用性 SLO (28d)
let window = 28d;
requests
| where timestamp > ago(window)
| where url startswith "https://api.example.com/v1/"
| extend success = case(
    resultCode startswith "2", true,
    resultCode startswith "3", true,
    resultCode startswith "4" and resultCode != "500", true, // 4xx はクライアント起因
    false
)
| summarize 
    total = count(),
    successful = countif(success == true)
| extend availability_rate = round(successful * 100.0 / total, 4)
```

---

## 5. SLO レビューサイクル

| イベント | 頻度 | 参加者 | 議題 |
|---------|------|-------|------|
| デイリー | 毎日 | SRE | バーンレート確認 |
| ウィークリー | 週次 | SRE + 開発リード | エラーバジェット消費・トレンド |
| マンスリー | 月次 | SRE + プロダクトオーナー | SLO 達成度・改善優先度 |
| クォータリー | 四半期 | SRE + 経営層 | SLO 目標値見直し |

---

## 6. インシデント時対応

```
[アラート発火]
    ↓
[SRE が一次受け]
    ↓
[インシデントレベル判定: P1〜P4]
    ↓
[ランブック実行] ← `ランブック規約.md` 参照
    ↓
[復旧 → エラーバジェット消費記録]
    ↓
[Postmortem (5営業日以内)]
    ↓
[SLO ダッシュボード反映]
```

---

## 7. SLO テンプレート (新規追加用)

```yaml
# SLO 新規追加テンプレート
sli_name: "[埋めてください]"
description: "[このSLIが何を計測するか]"
formula: "[数式]"
exclusions: "[除外条件]"
window: "rolling 28-day"
slo_target: "[X.X%]"
error_budget: "[計算結果]"
measurement_tool: "[Azure Monitor / Datadog / etc]"
dashboard_url: "[URL]"
owner: "[役職 + 氏名]"
review_frequency: "[monthly]"
```

---

## 8. 変更履歴

| 日付 | 版 | 変更内容 | 変更者 |
|------|----|---------|-------|
| YYYY-MM-DD | 1.0 | 初版作成 | [...] |

---

## 9. pm-blueprint 連携

- **出力レイヤー:** Layer 4 (NFR: Reliability/Performance) / Layer 6 (Operations)
- **関連品質ゲート ID:**
  - NFR-SLO-001 (High) ... 本書の存在 + ダッシュボード URL
  - NFR-RTORPO-001 ... DR 設計書との整合
  - DOC-RUNBOOK-001 ... ランブック規約との整合
- **ゲート通過条件:**
  1. 主要サービスごとに SLI/SLO が定義されている
  2. エラーバジェットとバーンレートアラートが設定済
  3. ダッシュボード URL が機能している (404 でない)
  4. 直近 1 四半期以内にレビュー実施記録あり
- **連携 ADR:** SLO 目標変更や計測方法変更は ADR 化
