---
name: RTO_RPO_DR設計
description: 災害復旧 (DR) の RTO / RPO 目標、Active-Active / Active-Passive / Pilot Light の選定、演習頻度、Genesys SaaS の責任分界点、Azure Japan East/West 冗長を設計するスキル。品質ゲート NFR-RTORPO-001 に対応。
---

# RTO / RPO / DR (Disaster Recovery) 設計

## 対応する品質ゲート

| ID | 観点 | 既定の Conditional 条件 |
|----|------|------|
| NFR-RTORPO-001 | RTO / RPO と DR | RTO / RPO の数値目標、DR 構成、演習頻度、SaaS 責任分界点が宣言されていない |

## 概要

DR (災害復旧) の議論は曖昧になりがち。**3 つの数字** で握る:

- **RTO (Recovery Time Objective)** : 復旧までの目標時間
- **RPO (Recovery Point Objective)** : 失っても良いデータの最大時間幅
- **演習頻度** : 復旧シナリオを年何回・どの規模で実施するか

そして、**SaaS (Genesys / Salesforce / Snowflake) との責任分界** を明示しないと、本番障害時に「これは Genesys の領域」「いやウチの領域」で時間を浪費する。

## いつ使うか

- 本番リリース前 (BCP / DRP の確定)
- 監査 (J-SOX, ISMS 27031) で問われた時
- 重大障害の RCA で「もっと早く復旧できた」と判明した時
- マルチリージョン投資の意思決定時

## 手順 (5 ステップ)

### ① RTO / RPO を業務影響度から逆算する

#### MTPD / RTO / RPO の関係

```
事故発生 ←─── 経過時間 ───→
   |
   ├─ RPO (例 5 分前) … 失う最大データ量
   |
   ├──────────────────── 復旧開始
   |
   ├──────────── RTO (例 4h) ── 復旧完了 (許容上限)
   |
   └──────────── MTPD (例 8h) ── これを超えると事業継続不可
```

- **MTPD (Maximum Tolerable Period of Disruption)** : 事業継続不可となる上限
- **RTO** : MTPD より十分小さい時間 (通常 MTPD の半分以下)
- **RPO** : 失っても許せるデータ更新の時間幅

#### 業務クラス別の標準値

| 業務クラス | 例 | MTPD | RTO | RPO |
|----------|------|------|-----|-----|
| Mission Critical | 決済、医療 | 4h | 1h | 0 (同期) |
| Business Critical | コールセンター受付 | 8h | 4h | 5 min |
| Important | BI / レポート | 24h | 8h | 1h |
| Standard | 内部ツール | 72h | 24h | 24h |

#### 宣言テンプレート

```yaml
rto_rpo_targets:
  - service: 顧客問合せ受付 API
    business_class: Business Critical
    mtpd: 8h
    rto: 4h
    rpo: 5min
    rationale: |
      Genesys からの受信を 5 分ロストしたら問合せ 50 件喪失、
      4h 停止で他チャネル誘導が可能だが顧客満足度に影響大
  - service: VoC 分析 (Bronze→Gold)
    business_class: Important
    mtpd: 48h
    rto: 24h
    rpo: 4h
  - service: SAP 仕訳バッチ
    business_class: Important
    mtpd: 24h (T+1 締めに合わせる)
    rto: 8h
    rpo: 1日 (日次バッチなのでほぼ同等)
```

### ② DR 戦略を選定する (NFR-RTORPO-001)

| 戦略 | 概要 | RTO 目安 | RPO 目安 | コスト |
|-----|------|---------|---------|------|
| **Backup & Restore** | バックアップから新規構築 | 8〜24h | 24h | 低 |
| **Pilot Light** | DR 側に最小限のリソース常駐、データだけ複製 | 1〜4h | 1h〜数分 | 中 |
| **Warm Standby** | DR 側でも一部稼働、本番より低スペック | 15〜60 min | 数分 | 高 |
| **Active-Passive (Hot Standby)** | 同スペックで待機、フェイルオーバー時切替 | 5〜15 min | 数秒〜数分 | 高 |
| **Active-Active** | 両系で常時負荷分散、片系障害でもう一方が全量受ける | < 5 min | 0 (同期) | 最高 |
| **Multi-Region Active-Active** | 地理的に離れた複数リージョンで Active-Active | < 5 min | 0 | 最高 |

#### 選定マトリクス (RTO / RPO 達成可否)

| 戦略 | 1h RTO 達成 | 5 min RPO 達成 |
|------|-----------|--------------|
| Backup & Restore | × | × |
| Pilot Light | △ | ○ |
| Warm Standby | ○ | ○ |
| Active-Passive | ○ | ○ |
| Active-Active | ◎ | ◎ |

### ③ Azure Japan East / West の活用パターン

#### Azure リージョンペア (推奨)

| Primary | Paired Region | 同期距離 | 用途 |
|---------|------------|---------|-----|
| Japan East (東京) | Japan West (大阪) | 約 400km | 国内 BCP の標準 |
| East US 2 | Central US | 国内代替 | 海外向けに選択 |

#### Azure サービス別 DR 構成例

| サービス | Active-Passive | Active-Active |
|---------|--------------|--------------|
| Azure DB for PostgreSQL Flex | Geo-redundant Backup + Read Replica | (将来 Hyperscale 検討) |
| Azure Container Apps | East 主 / West 待機 (Front Door 切替) | East/West 両系 + Front Door 振分 |
| Azure Blob | RA-GZRS (読込地理冗長) | Geo-Redundant + Replication ルール |
| Azure Key Vault | 自動 Geo-replicated | (Default で冗長) |
| Azure Event Hubs | Geo-DR Pairing (Active-Passive) | (Active-Active は要設計) |
| Databricks | Workspace を East/West 両方に。Delta Lake は Storage 経由で複製 | (高コスト) |
| Application Insights | Workspace Linked, Diagnostic Setting | - |

#### 構成図 (推奨: Active-Passive + Front Door)

```
                       [Azure Front Door (Premium)]
                                  ↓
                  ┌──────────────┼──────────────┐
                  ↓                              ↓
         [Japan East (Primary)]        [Japan West (Standby)]
         - Container Apps (running)     - Container Apps (idle, 1 instance)
         - Postgres Flex (Primary)      - Postgres Flex (Read Replica)
         - Storage (RA-GZRS)            - Storage (Read access only)
         - Event Hubs                   - Event Hubs (Geo-DR pair)
         - Databricks Workspace (Live)  - Databricks Workspace (Idle)
                  ↓                              ↑
                  └────── Async Replication ────┘
```

#### 切替 (Failover) 手順

```yaml
failover_runbook:
  manual_failover_steps:
    1. 障害確認 (Azure Service Health, Front Door Probe)
    2. SRE Lead が Failover 判断 (RTO 30 min 以内)
    3. Postgres Read Replica を Promote (Auto-Failover Group)
    4. Front Door の Backend Priority を切替
    5. Event Hubs Geo-DR Failover 実行 (az eventhubs geodr fail-over)
    6. アプリ DNS / Webhook 受信エンドポイント切替
    7. Databricks ジョブを West Workspace で起動
    8. ステータスページ更新、顧客通知
  automated_steps:
    - Front Door Health Probe による自動切替 (60 秒以内)
  rollback:
    - East 復旧後、Replica を再構成して再同期 (数時間)
    - 業務影響時間 → ポストモーテム
```

### ④ Genesys Cloud SaaS の責任分界点

Genesys Cloud は SaaS のため、**自社 DR 範囲は契約境界の手前まで**。

#### 責任マトリクス (RACI 風)

| コンポーネント | Genesys 責任 | 自社責任 |
|------------|----------|--------|
| 通話基盤 (SBC, Media Server) | ◎ | - |
| ACD / IVR | ◎ | - |
| Recording Storage | ◎ (S3 互換) | バックアップ要否判断 |
| Public API | ◎ (SLA 99.99%) | リトライ・冪等性 |
| Webhook 配信 | ◎ (at-least-once) | 受信・冪等処理・DLQ |
| 受信側 Webhook サーバー | - | ◎ (RTO/RPO 設計) |
| 通話ログ取り込み (after API) | - | ◎ |
| 業務 DB (Postgres) | - | ◎ |
| 分析 (Databricks) | - | ◎ |

#### Genesys 障害時の自社対応

```yaml
genesys_outage_response:
  detection:
    - Status Page 監視 (genesys.com/status RSS)
    - Webhook 受信途絶検知 (5 分間ゼロで Alert)
  containment:
    - Inbox テーブルに「Genesys 障害区間」マーク
    - 顧客向けに代替問合せ手段案内 (メールフォーム)
  recovery:
    - Genesys 復旧後、API で過去 24h の Interaction を Bulk Pull
    - Inbox 突合 (重複排除済み = OK、未受信 = Recovery 対象)
  communication:
    - 受信再開後、業務部門に「失われた可能性のある時間帯」を報告
```

#### SaaS 全般の責任分界 (Salesforce / Snowflake / SAP も同様)

```yaml
saas_dr_boundary:
  - vendor: Genesys Cloud
    saas_sla: 99.99%
    our_dr_for: Webhook 受信・Inbox・業務 DB
  - vendor: Salesforce
    saas_sla: 99.9%
    our_dr_for: API 連携キュー、ローカル CRM スナップショット
  - vendor: Snowflake
    saas_sla: 99.9%
    our_dr_for: 連携バッチ、メタデータバックアップ
  - vendor: SAP S/4HANA Cloud
    saas_sla: 99.7%
    our_dr_for: IDOC キュー、ファイル中継
```

### ⑤ DR 演習 (DR Drill) を計画する

「DR 構成があっても、演習しないと本番では動かない」が鉄則。

#### 演習タイプ

| タイプ | 概要 | 頻度 |
|------|------|-----|
| Tabletop (机上) | シナリオ読合わせ、判断練習 | 四半期 |
| Walkthrough | 手順を実環境でなぞる (実切替なし) | 四半期 |
| Simulation | DR 環境で疑似障害を起こす (本番影響なし) | 半年 |
| Full Failover | 本番→DR 切替を実施し、業務時間外で運用 | **年 1 回必須** |
| Chaos Engineering | ランダムに本番ノードを停止 | 月次 (成熟組織のみ) |

#### 演習計画テンプレート

```yaml
dr_drill_plan:
  annual_full_failover:
    schedule: 毎年 11 月 第 2 土曜 02:00 JST
    duration: 4h (切替 1h + 運用 2h + 切戻 1h)
    scope:
      - Azure Region (East → West)
      - Postgres / Container Apps / Storage / Event Hubs
    success_criteria:
      - RTO < 1h
      - データロス 0 件
      - 業務シナリオ Top 5 が DR 環境で完了
    ngo_exit_criteria:
      - RTO 4h 超過 → Drill 中止、本番に切戻
  quarterly_tabletop:
    scenarios:
      - "Japan East 全域停止 (Azure Region 障害)"
      - "Postgres Primary が応答停止"
      - "Genesys Cloud が 4h 停止"
      - "Webhook 受信率が突然 0% になる"
  monthly_chaos:
    scope: dev / stg のみ
    tool: Chaos Studio (Azure)
```

#### 演習結果の記録

```yaml
drill_record:
  drill_id: DRILL-2026-04
  date: 2026-04-12
  type: Full Failover
  results:
    rto_achieved: 52 min (target 1h)
    rpo_achieved: 0 sec (sync replication 動作)
    issues_found:
      - Front Door Probe 検出が 90 秒遅延 (要 Probe 設定見直し)
      - Databricks Workspace 起動に 8 min かかる (Pre-warm 検討)
    follow_up:
      - ADR-0050: Front Door Probe interval を 30s → 10s
      - ADR-0051: Databricks West Workspace に Cluster Pool 設置
```

## 出力フォーマット

```
docs/
├── reliability/
│   ├── 05_rto_rpo_targets.yaml
│   ├── 06_dr_strategy.md
│   ├── 07_failover_runbook.md
│   ├── 08_saas_boundaries.yaml
│   └── 09_drill_plan.yaml
├── adr/
│   └── 0050-dr-strategy.md
runbooks/
└── failover/
    ├── postgres-failover.md
    ├── frontdoor-failover.md
    └── eventhubs-geodr.md
```

## 例示: コールセンター VoC 基盤の DR 構成

```yaml
example_dr:
  primary_region: Japan East
  paired_region: Japan West
  strategy: Active-Passive (Warm Standby)
  rto_target: 1h
  rpo_target: 5 min
  components:
    front_door: Active-Active (両 Region への振分)
    container_apps:
      east: 3 replicas (Active)
      west: 1 replica (Idle, Pre-warmed)
    postgres:
      east: Primary
      west: Read Replica (Auto-Failover Group)
    storage: RA-GZRS (Read Access Geo-Zone-Redundant)
    eventhubs: Geo-DR Pairing
    databricks: East Workspace (Active) + West Workspace (Cold)
  cost_impact:
    primary_only: ¥1,500,000/month
    with_dr: ¥1,950,000/month  (+30%)
  drill_schedule:
    full_failover: annual (Nov 第 2 土曜)
    tabletop: quarterly
    chaos_dev: monthly
```

## 作成時のチェックリスト

- [ ] サービスごとに RTO/RPO の数値目標が宣言されている
- [ ] 業務クラス (Mission/Business/Important/Standard) が割り当てられている
- [ ] DR 戦略 (Backup/Pilot/Warm/Active-Passive/Active-Active) が選定済み
- [ ] Azure リージョンペア (Japan East/West 等) が選定されている
- [ ] Front Door / Auto-Failover Group などの自動切替メカニズムが設計されている
- [ ] Failover Runbook が手順化されている
- [ ] SaaS (Genesys / Salesforce / Snowflake / SAP) の責任分界が明記されている
- [ ] SaaS 障害時の自社対応手順がある
- [ ] 年 1 回の Full Failover 演習が計画されている
- [ ] 四半期 Tabletop 演習が計画されている
- [ ] 演習結果記録テンプレートが整備されている
- [ ] DR コスト追加分が予算化されている
- [ ] ADR (DR 戦略) が起票済み

## pm-blueprint 連携

| 既存サブスキル | 連携ポイント |
|--------------|-----------|
| `layer-4-requirements/SLI_SLO_エラーバジェット.md` | SLO Floor (99.0%) を下回ると DR 発動判断 |
| `layer-4-requirements/SMART非機能要件.md` | Reliability NFR の Outstanding 水準を DR 設計と整合 |
| `layer-4-requirements/可観測性三本柱.md` | DR 切替時の log/metric/trace を統合 |
| `layer-3-architecture/環境分離_IaC.md` | Region 跨ぎの IaC モジュール設計 |
| `layer-3-architecture/データ設計詳細.md` | RPO 目標から非同期/同期レプリケーション選択 |

## 品質ゲート対応サマリ

| ゲート ID | 本スキル該当節 | 出力アーティファクト |
|----------|--------------|------------------|
| NFR-RTORPO-001 | 手順 ①〜⑤ | `docs/reliability/{05..09}` + ADR-0050 |

## 参考

- ISO/IEC 22301 (BCMS) / ISO/IEC 27031
- NIST SP 800-34 "Contingency Planning Guide"
- Azure Architecture Center "Disaster Recovery": https://learn.microsoft.com/en-us/azure/architecture/framework/resiliency/backup-and-recovery
- Microsoft Learn "Cross-region disaster recovery for Azure DB for PostgreSQL"
- Azure Site Recovery / Azure Backup Documentation
- Genesys Cloud Service Status: https://status.mypurecloud.com
- Google SRE Workbook "Managing risk", "Postmortem culture"
- AWS DR White Paper "Disaster Recovery of Workloads on AWS"
