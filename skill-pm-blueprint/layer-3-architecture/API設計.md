---
name: API設計
description: REST / GraphQL / gRPC の選定、Idempotency-Key 仕様、バージョニング戦略、OpenAPI 3.x テンプレートを体系的に設計するスキル。品質ゲート API-IDEMPOTENCY-001 / API-STYLE-001 / API-VERSION-001 に対応。
---

# API 設計 (Style / Idempotency / Versioning)

## 対応する品質ゲート

| ID | 観点 | 既定の Fail/Conditional 条件 |
|----|------|------|
| API-STYLE-001 | API スタイル選定 | REST / GraphQL / gRPC のどれを使うか、根拠付きで選定されていない |
| API-IDEMPOTENCY-001 | 冪等性 | 副作用ある操作 (POST/PUT/DELETE) に Idempotency-Key 設計がない |
| API-VERSION-001 | バージョニング | 破壊的変更時の互換戦略 (path / header / media type) が宣言されていない |

## 概要

API は **「契約」** である。一度公開すると簡単には変えられない。

本スキルでは、矢羽パターンの ③ 詳細設計で:

1. **スタイル選定** (REST / GraphQL / gRPC / WebSocket / Webhook)
2. **冪等性設計** (Idempotency-Key, リトライ前提)
3. **バージョニング戦略** (path / header / media type)
4. **OpenAPI 3.x で契約を明文化**

の 4 点を設計し、ADR と合わせて記録する。

## いつ使うか

- 外部公開 API、社内サービス間 API、フロントエンド向け BFF を設計する時
- Genesys / SAP / Databricks / Azure OpenAI 等の外部 API を呼ぶラッパー層を作る時
- 既存 API の v2 / 後継 API を企画する時
- パートナー企業に API を提供する時 (SLA, 課金, レート制限が絡む)

## 手順 (4 ステップ)

### ① API スタイルを選定する (API-STYLE-001)

選定マトリクス:

| スタイル | 強み | 弱み | 採用適性 |
|---------|-----|-----|--------|
| **REST + JSON** | 標準・学習容易・キャッシュ効く | 過剰取得 / N+1 問題 | 公開 API、CRUD 中心、シンプルな業務系 |
| **GraphQL** | 必要な項目だけ取得・型安全 | キャッシュ難・N+1 慎重対応 | フロント主導、画面ごとに必要項目が違う BFF |
| **gRPC (Protobuf)** | 高速・型強力・ストリーミング | ブラウザ直接利用しにくい | サービス間通信、内部マイクロサービス |
| **WebSocket / SSE** | 双方向 / Push | 接続管理コスト | 通話メーター、リアルタイム通知 |
| **Webhook** | サーバーから受動的に呼ぶ | 受信側の冪等性必須 | 外部 SaaS (Genesys) からの通知受信 |
| **Batch (CSV/Parquet)** | 大量データ処理 | リアルタイム性ゼロ | 日次バッチ、SAP 仕訳取込 |

**選定の問い:**

- 公開対象 (Public / Partner / Internal) はどこ?
- 1 リクエストで取る項目は固定的か可変的か?
- レイテンシ予算は?
- ストリーミング (双方向) が必要か?
- クライアントは Web / モバイル / サーバー / IoT?

**コールセンター実例:**

```yaml
api_styles:
  - service: 顧客問合せ管理 (内部マイクロサービス)
    style: gRPC + Protobuf
    rationale: 社内サービス間、レイテンシ予算 50ms、型安全重視
  - service: 顧客向け問合せ受付 Web
    style: REST + JSON (HTTPS)
    rationale: 公開、フロント実装容易、CDN キャッシュ可能
  - service: Genesys からの通話イベント受信
    style: Webhook (HTTPS POST)
    rationale: Genesys から Push、HMAC 署名で認証、Inbox で冪等処理
  - service: BI / 管理画面
    style: GraphQL (BFF)
    rationale: 画面ごとに表示項目が異なる、Apollo Client で型安全
  - service: SAP 仕訳連携
    style: Batch (Parquet on Blob)
    rationale: 日次 1 回、大量レコード、SAP IDOC 互換要件
```

### ② Idempotency (冪等性) を設計する (API-IDEMPOTENCY-001)

**鉄則:** ネットワークは必ず失敗する。クライアントは必ずリトライする。サーバーは「同じ操作を 2 回受けても 1 回扱い」にしなければならない。

#### 冪等な HTTP メソッド

| メソッド | 仕様上の冪等性 | 運用での扱い |
|---------|--------------|-----------|
| GET | 冪等 | 副作用なしを徹底 |
| PUT | 冪等 | 全置換、`If-Match` で楽観的ロック |
| DELETE | 冪等 | 「存在しない」も 200 / 204 |
| **POST** | **非冪等** | **Idempotency-Key 必須** |
| PATCH | 非冪等 (基本) | Idempotency-Key 必須 |

#### Idempotency-Key 標準仕様 (Stripe / IETF Draft 準拠)

**リクエストヘッダ:**

```
POST /v1/interactions
Idempotency-Key: 7c9e6679-7425-40de-944b-e07fc1f90ae7
Content-Type: application/json
```

**サーバー側ロジック:**

1. `Idempotency-Key` を受け取る (UUID v4 / ULID 推奨, クライアント生成)
2. キャッシュ (Redis 推奨) に `key + リクエストハッシュ` を保存。TTL 24h〜7d
3. 同じキーで同じ内容なら **保存済みレスポンスを返す** (新規処理しない)
4. 同じキーで内容が違うなら **`409 Conflict` を返す**
5. 進行中なら **`409 Conflict (in_progress)` を返す**

**実装例 (擬似コード):**

```python
def post_interaction(idempotency_key, payload):
    cache_key = f"idempotency:{idempotency_key}"
    request_hash = sha256(canonical_json(payload))

    cached = redis.get(cache_key)
    if cached:
        if cached["hash"] != request_hash:
            return 409, {"error": "idempotency_key_mismatch"}
        if cached["status"] == "in_progress":
            return 409, {"error": "idempotency_in_progress"}
        return cached["status_code"], cached["body"]

    # 進行中マーカー
    redis.setex(cache_key, 24*3600, {"status": "in_progress", "hash": request_hash})

    try:
        result = create_interaction(payload)
        redis.setex(cache_key, 7*24*3600, {
            "status": "completed",
            "hash": request_hash,
            "status_code": 201,
            "body": result,
        })
        return 201, result
    except Exception as e:
        redis.delete(cache_key)
        raise
```

#### Idempotency-Key の宣言テンプレート

```yaml
idempotency:
  - endpoint: POST /v1/interactions
    key_header: Idempotency-Key
    key_format: UUID v4 / ULID (クライアント生成)
    storage: Redis (TTL 7 日)
    response_caching: 全レスポンス (status_code + body) を保存
    conflict_status: 409 Conflict (key_mismatch / in_progress)
  - endpoint: PATCH /v1/interactions/{id}/tags
    key_header: Idempotency-Key
    natural_idempotency: false (タグ追加は非冪等)
    storage: Redis (TTL 24h)
```

**鉄則:**

- POST / PATCH には **常に** Idempotency-Key を要求する
- TTL は最低 24h、決済や仕訳など重要なものは 7 日
- レスポンスごとキャッシュする (status code, headers, body すべて)

### ③ バージョニング戦略を選定する (API-VERSION-001)

3 種類の戦略から 1 つ選び、組織標準として ADR で固定する。

| 方式 | 例 | 強み | 弱み |
|-----|------|-----|------|
| **URL Path** | `/v1/interactions` `/v2/interactions` | 視認性 / キャッシュ単純 | クライアント側 URL 書き換え必要 |
| **Header** | `Accept-Version: 2` | URL 不変 / キレイ | デバッグしにくい・キャッシュ難 |
| **Media Type** | `Accept: application/vnd.acme.v2+json` | RESTful 純度高 | 普及度低・実装難 |

**推奨:** **URL Path** (パートナー公開時)、**Header** (内部のみ・実験的バージョン)

#### 互換性ポリシー

| 変更タイプ | 互換性 | バージョン上げ |
|----------|-------|------------|
| フィールド追加 (任意) | 後方互換 | 不要 |
| フィールド削除 | 破壊的 | **必須** |
| フィールド型変更 | 破壊的 | **必須** |
| エンドポイント削除 | 破壊的 | **必須** |
| エラーコード追加 | グレー | 推奨 |
| レート制限変更 | グレー | 告知 30 日前 |

#### Deprecation Policy (廃止ポリシー)

```yaml
deprecation_policy:
  notification: 廃止 6 ヶ月前 (Email + Changelog + Header)
  sunset_header: |
    Sunset: Sat, 31 Dec 2026 23:59:59 GMT
    Deprecation: true
    Link: <https://api.acme.com/docs/v2-migration>; rel="deprecation"
  parallel_run: 旧 v1 と新 v2 を最低 12 ヶ月並走
  metrics: v1 残存利用率を月次計測、5% 未満で廃止判定
```

#### バージョン宣言テンプレート

```yaml
versioning:
  scheme: url_path
  current: v1
  next: v2 (planned 2026-Q4)
  compatibility_promise:
    - additive_only_within_major: true
    - removal_requires_major_bump: true
    - deprecation_notice_period_days: 180
  internal_apis_scheme: header (Accept-Version)
```

### ④ OpenAPI 3.x で契約を明文化する

OpenAPI YAML を **Single Source of Truth** とし、コード・ドキュメント・モックは全てここから生成する。

#### 最小テンプレート

```yaml
openapi: 3.1.0
info:
  title: Customer Interaction API
  version: 1.2.0
  description: |
    顧客問合せ受付 / 履歴 / 分析結果 API
    変更履歴: docs/api-changelog.md
  contact:
    name: API Platform Team
    email: api-platform@acme.co.jp

servers:
  - url: https://api.acme.co.jp/v1
    description: Production
  - url: https://api-stg.acme.co.jp/v1
    description: Staging

security:
  - bearerAuth: []

paths:
  /interactions:
    post:
      summary: 問合せレコードを新規作成
      operationId: createInteraction
      parameters:
        - in: header
          name: Idempotency-Key
          required: true
          schema:
            type: string
            format: uuid
          description: クライアント生成。重複送信防止のため必須
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/InteractionCreate'
      responses:
        '201':
          description: 作成成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Interaction'
        '409':
          description: 冪等性キー重複
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '422':
          description: バリデーションエラー
        '429':
          description: レート制限超過
          headers:
            Retry-After:
              schema:
                type: integer

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    InteractionCreate:
      type: object
      required: [customer_id, channel, started_at]
      properties:
        customer_id:
          type: string
          format: uuid
        channel:
          type: string
          enum: [voice, chat, email]
        started_at:
          type: string
          format: date-time
    Interaction:
      allOf:
        - $ref: '#/components/schemas/InteractionCreate'
        - type: object
          properties:
            interaction_id:
              type: string
              format: uuid
            created_at:
              type: string
              format: date-time
    Error:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
          example: idempotency_key_mismatch
        message:
          type: string
        request_id:
          type: string
```

#### 必須要素チェックリスト

- [ ] `info.version` (Semantic Versioning)
- [ ] `servers` (env ごとに記載)
- [ ] `security` (Bearer / API Key / mTLS)
- [ ] 全 endpoint に `operationId` (SDK 生成キー)
- [ ] エラースキーマ統一 (`Error` を `$ref` で再利用)
- [ ] `4xx` / `5xx` の代表例を記載
- [ ] レート制限 (`429 Retry-After`)
- [ ] Idempotency-Key パラメータ (POST / PATCH)
- [ ] Pagination パターン (cursor / offset)
- [ ] Filtering / Sorting 規約

## 出力フォーマット

```
docs/
├── api/
│   ├── openapi.yaml           # SoT
│   ├── changelog.md
│   ├── style-decision.md      # API-STYLE-001 の選定根拠
│   ├── idempotency-spec.md    # API-IDEMPOTENCY-001
│   └── versioning-policy.md   # API-VERSION-001
└── adr/
    ├── 0020-api-style.md
    ├── 0021-idempotency.md
    └── 0022-versioning.md
```

## 例示: ABC社 VoC API

| エンドポイント | スタイル | 冪等性 | バージョン |
|------------|--------|------|---------|
| POST /v1/voc/inquiries | REST | Idempotency-Key 必須 | URL path |
| GET /v1/voc/inquiries/{id} | REST | (GET) | URL path |
| POST /webhooks/genesys | Webhook | HMAC + interaction_id 重複排除 | URL path |
| (gRPC) VocAnalysis.Classify | gRPC | request_id 冪等 | proto package version |

## 作成時のチェックリスト

- [ ] API スタイル (REST / GraphQL / gRPC / Webhook / Batch) が選定され ADR がある
- [ ] 公開範囲 (Public / Partner / Internal) が宣言されている
- [ ] 全副作用エンドポイントに Idempotency-Key 仕様がある
- [ ] Idempotency キャッシュの TTL と保管先 (Redis 等) が宣言されている
- [ ] バージョニング戦略 (path / header / media type) が選定されている
- [ ] 後方互換ポリシーと Deprecation 期間が明記されている
- [ ] OpenAPI 3.x が SoT として整備されている
- [ ] エラースキーマが統一されている (code, message, request_id)
- [ ] レート制限 (`429 Retry-After`) が定義されている
- [ ] Pagination 規約 (cursor / offset) が統一されている
- [ ] 関連 ADR (style / idempotency / versioning) が起票されている

## pm-blueprint 連携

| 既存サブスキル | 連携ポイント |
|--------------|-----------|
| `layer-3-architecture/データ設計詳細.md` | Idempotency-Key の保管先 (Redis) と Outbox は同 Tx で実装 |
| `layer-3-architecture/ADR作成.md` | スタイル選定 / バージョニング戦略 / Idempotency 仕様は ADR (Type 1) |
| `layer-4-requirements/SMART非機能要件.md` | レイテンシ・スループット・レート制限は NFR ランディングゾーン |
| `layer-4-requirements/SLI_SLO_エラーバジェット.md` | API SLI (P95 latency, error rate) を 99.9% SLO で計測 |
| `layer-4-requirements/可観測性三本柱.md` | request_id を Trace ID として Log/Metric/Trace に伝播 |

## 品質ゲート対応サマリ

| ゲート ID | 本スキル該当節 | 出力アーティファクト |
|----------|--------------|------------------|
| API-STYLE-001 | 手順 ① | `docs/api/style-decision.md` + ADR-0020 |
| API-IDEMPOTENCY-001 | 手順 ② | `docs/api/idempotency-spec.md` + ADR-0021 |
| API-VERSION-001 | 手順 ③ | `docs/api/versioning-policy.md` + ADR-0022 |

## 参考

- IETF "The Idempotency-Key HTTP Header Field" (Draft, Stripe 提案)
- Stripe API Reference (Idempotency 実装の事実上の標準)
- Microsoft REST API Guidelines: https://github.com/microsoft/api-guidelines
- Zalando RESTful API Guidelines: https://opensource.zalando.com/restful-api-guidelines/
- OpenAPI Specification 3.1.0: https://spec.openapis.org/oas/v3.1.0
- Roy Fielding, "Architectural Styles" (REST 原典)
- gRPC Best Practices (Google Cloud)
