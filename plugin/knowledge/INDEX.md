# Agentic Quality Gate — Knowledge Index

全 176 エントリ（status=active のみ）。skip 0 件（draft/deprecated）。

**Master**: `knowledge/master.xlsx`（編集元）
**Generated**: このファイルと配下 *.md は master.xlsx から自動生成（`scripts/excel_to_md.py`）

## P0 構想・要件定義（13件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [LEGAL-INDUSTRY-001](phases/P0/LEGAL-INDUSTRY-001.md) | 🔴 critical | 対象システムにおいて、業法該当性（金融/医療/保険/運送/不動産/通信）が法務承認を受け文書化されているか | `legal/industry` |
| [LEGAL-PII-001](phases/P0/LEGAL-PII-001.md) | 🔴 critical | 取扱う個人情報の定義・利用目的・第三者提供の有無が明文化され、個人情報保護法および社内規程に整合しているか | `legal/privacy` |
| [LEGAL-TOS-001](phases/P0/LEGAL-TOS-001.md) | 🔴 critical | 対象システムにおいて、データ取得元の利用規約・スクレイピング適法性が法務承認を受け文書化されているか | `legal/data` |
| [COST-ESTIMATE-001](phases/P0/COST-ESTIMATE-001.md) | 🟠 high | 概算コスト試算（クラウド/LLM/データ）が事前に試算され、ハードリミット・アラートが設定されているか | `cost/estimation` |
| [COST-UNIT-001](phases/P0/COST-UNIT-001.md) | 🟠 high | 課金単位（req/token/storage/transfer）理解が事前に試算され、ハードリミット・アラートが設定され | `cost/unit` |
| [LEGAL-AIGEN-001](phases/P0/LEGAL-AIGEN-001.md) | 🟠 high | 対象システムにおいて、AI生成物の著作権・学習利用・侵害可能性が法務承認を受け文書化されているか | `legal/copyright` |
| [LEGAL-CROSSBORDER-001](phases/P0/LEGAL-CROSSBORDER-001.md) | 🟠 high | 対象システムにおいて、越境データ移転（GDPR/米輸出規制/中国データ三法）が法務承認を受け文書化されているか | `legal/crossborder` |
| [LEGAL-DATACLASS-001](phases/P0/LEGAL-DATACLASS-001.md) | 🟠 high | データ設計としてデータ分類（公開/内部/機密/極秘）の宣言が文書化され、レビュー承認されているか | `data/classification` |
| [LEGAL-LICENSE-001](phases/P0/LEGAL-LICENSE-001.md) | 🟠 high | 対象システムにおいて、OSSライセンス（GPL/AGPL汚染、商用可否）が法務承認を受け文書化されているか | `legal/license` |
| [LEGAL-THREATMODEL-001](phases/P0/LEGAL-THREATMODEL-001.md) | 🟠 high | 対象コード・設定においてSTRIDE/LINDDUN による脅威モデル v0に対応する防御機構が実装され、検証されている | `security/threatmodel` |
| [STRATEGY-LIFETIME-001](phases/P0/STRATEGY-LIFETIME-001.md) | 🟠 high | プロジェクトとして「使い捨て or 恒久」宣言と品質投資の整合が明文化され、関係者に共有されているか | `strategy/lifetime` |
| [COST-FREETIER-001](phases/P0/COST-FREETIER-001.md) | 🟡 medium | 無料枠終了後・トライアル切れ挙動が事前に試算され、ハードリミット・アラートが設定されているか | `cost/freetier` |
| [LEGAL-PATENT-001](phases/P0/LEGAL-PATENT-001.md) | 🟡 medium | 対象システムにおいて、特許侵害リスク事前調査が法務承認を受け文書化されているか | `legal/patent` |

## P1 アーキテクチャ・データ設計（35件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [API-IDEMPOTENCY-001](phases/P1/API-IDEMPOTENCY-001.md) | 🔴 critical | API設計として冪等性キーヘッダ設計が仕様化され、設計レビュー承認されているか | `api/idempotency` |
| [ARCH-LAYER-001](phases/P1/ARCH-LAYER-001.md) | 🔴 critical | アーキテクチャ決定としてIaaS/PaaS/SaaSレイヤ選定とリスク受容がADRに記録され承認されているか | `architecture/layer` |
| [DATA-ENTITY-001](phases/P1/DATA-ENTITY-001.md) | 🔴 critical | データ設計としてエンティティ・関係性の言語化（多対多含む）が文書化され、レビュー承認されているか | `data/modeling` |
| [DATA-TX-001](phases/P1/DATA-TX-001.md) | 🔴 critical | データ設計としてトランザクション境界の宣言が文書化され、レビュー承認されているか | `data/transaction` |
| [ENV-SEPARATE-001](phases/P1/ENV-SEPARATE-001.md) | 🔴 critical | 環境設計としてdev/stg/prod 環境分離（DB含む）が実装され、構成ドリフトが検知できるか | `env/separation` |
| [SEC-AUTHZ-DESIGN-001](phases/P1/SEC-AUTHZ-DESIGN-001.md) | 🔴 critical | 対象コード・設定において認可方式（RBAC/ABAC/ReBAC/OPA）選定に対応する防御機構が実装され、検証されてい | `security/authz` |
| [SEC-TENANT-001](phases/P1/SEC-TENANT-001.md) | 🔴 critical | 対象コード・設定においてマルチテナント分離戦略（DB/schema/RLS）に対応する防御機構が実装され、検証されている | `security/tenant` |
| [API-STYLE-001](phases/P1/API-STYLE-001.md) | 🟠 high | API設計としてREST/GraphQL/gRPC選定が仕様化され、設計レビュー承認されているか | `api/style` |
| [API-VERSION-001](phases/P1/API-VERSION-001.md) | 🟠 high | API設計としてバージョニング戦略（path/header/media type）が仕様化され、設計レビュー承認されてい | `api/versioning` |
| [ARCH-AGGREGATE-001](phases/P1/ARCH-AGGREGATE-001.md) | 🟠 high | アーキテクチャ決定として集約境界とトランザクション境界の整合がADRに記録され承認されているか | `architecture/ddd` |
| [ARCH-DDD-001](phases/P1/ARCH-DDD-001.md) | 🟠 high | アーキテクチャ決定として境界づけられたコンテキスト・ユビキタス言語がADRに記録され承認されているか | `architecture/ddd` |
| [ARCH-MONOLITH-001](phases/P1/ARCH-MONOLITH-001.md) | 🟠 high | アーキテクチャ決定としてモノリス/モジュラーモノリス/マイクロサービス選定がADRに記録され承認されているか | `architecture/scale` |
| [ARCH-STYLE-001](phases/P1/ARCH-STYLE-001.md) | 🟠 high | レイヤード/ヘキサゴナル/オニオン/クリーンの中から選定したアーキテクチャスタイルと、その採用理由・トレードオフがADR | `architecture/style` |
| [DATA-CONSISTENCY-001](phases/P1/DATA-CONSISTENCY-001.md) | 🟠 high | データ設計としてACID vs BASE / 結果整合性の許容範囲が文書化され、レビュー承認されているか | `data/consistency` |
| [DATA-DELETE-001](phases/P1/DATA-DELETE-001.md) | 🟠 high | データ設計として物理削除 vs 論理削除の決定が文書化され、レビュー承認されているか | `data/modeling` |
| [DATA-DELIVERY-001](phases/P1/DATA-DELIVERY-001.md) | 🟠 high | メッセージング設計としてAt-least-once / At-most-once / Exactly-once 選定が文 | `messaging/delivery` |
| [DATA-HISTORY-001](phases/P1/DATA-HISTORY-001.md) | 🟠 high | データ設計として履歴保持戦略（イミュータブル/Event Sourcing）が文書化され、レビュー承認されているか | `data/history` |
| [DATA-OUTBOX-001](phases/P1/DATA-OUTBOX-001.md) | 🟠 high | データ設計としてOutboxパターンによる二重整合が文書化され、レビュー承認されているか | `data/outbox` |
| [DATA-SOT-001](phases/P1/DATA-SOT-001.md) | 🟠 high | データ設計としてSingle Source of Truth の宣言が文書化され、レビュー承認されているか | `data/governance` |
| [ENV-DATAFLOW-001](phases/P1/ENV-DATAFLOW-001.md) | 🟠 high | 環境設計として環境間のデータフロー禁則（本番→ステージは原則禁止）が実装され、構成ドリフトが検知できるか | `env/dataflow` |
| [NFR-OBS-001](phases/P1/NFR-OBS-001.md) | 🟠 high | 非機能要件として可観測性方針（log/metric/trace 三本柱）が合意・文書化され、計測手段が用意されているか | `nfr/observability` |
| [NFR-RTORPO-001](phases/P1/NFR-RTORPO-001.md) | 🟠 high | 非機能要件としてRTO/RPO 定義と DR 設計が合意・文書化され、計測手段が用意されているか | `nfr/dr` |
| [NFR-SLO-001](phases/P1/NFR-SLO-001.md) | 🟠 high | 非機能要件としてSLI/SLO/SLA とエラーバジェットが合意・文書化され、計測手段が用意されているか | `nfr/slo` |
| [PERF-SCALE-001](phases/P1/PERF-SCALE-001.md) | 🟠 high | 対象システムにおいて水平/垂直スケール戦略が抑止され、計測値が閾値内に収まっているか | `perf/scaling` |
| [PERF-STATELESS-001](phases/P1/PERF-STATELESS-001.md) | 🟠 high | 対象システムにおいてステートレス設計が抑止され、計測値が閾値内に収まっているか | `perf/stateless` |
| [SEC-CRYPTO-001](phases/P1/SEC-CRYPTO-001.md) | 🟠 high | 対象コード・設定において暗号化（保存時/転送時）と KMS 設計に対応する防御機構が実装され、検証されているか | `security/crypto` |
| [SEC-ZEROTRUST-001](phases/P1/SEC-ZEROTRUST-001.md) | 🟠 high | 対象コード・設定においてゼロトラスト前提の採用に対応する防御機構が実装され、検証されているか | `security/zerotrust` |
| [API-ERROR-001](phases/P1/API-ERROR-001.md) | 🟡 medium | API設計としてRFC7807 Problem Details 採用が仕様化され、設計レビュー承認されているか | `api/error` |
| [API-PAGE-001](phases/P1/API-PAGE-001.md) | 🟡 medium | API設計としてページネーション（offset/cursor）設計が仕様化され、設計レビュー承認されているか | `api/pagination` |
| [ARCH-EDA-001](phases/P1/ARCH-EDA-001.md) | 🟡 medium | アーキテクチャ決定としてイベント駆動・CQRS・イベントソーシング適用判断がADRに記録され承認されているか | `architecture/eda` |
| [ARCH-SERVERLESS-001](phases/P1/ARCH-SERVERLESS-001.md) | 🟡 medium | アーキテクチャ決定としてサーバーレス採用時の冷起動・実行時間制約がADRに記録され承認されているか | `architecture/serverless` |
| [DATA-NORMAL-001](phases/P1/DATA-NORMAL-001.md) | 🟡 medium | データ設計として正規化と非正規化のトレードオフが文書化され、レビュー承認されているか | `data/modeling` |
| [DATA-SAGA-001](phases/P1/DATA-SAGA-001.md) | 🟡 medium | データ設計として分散トランザクション（Saga）と補償処理が文書化され、レビュー承認されているか | `data/saga` |
| [PERF-CACHE-001](phases/P1/PERF-CACHE-001.md) | 🟡 medium | 対象システムにおいてキャッシュ層設計（aside/through/back）が抑止され、計測値が閾値内に収まっているか | `perf/cache` |
| [PERF-CAP-001](phases/P1/PERF-CAP-001.md) | 🟡 medium | 対象システムにおいてCAP/PACELC を踏まえた選択が抑止され、計測値が閾値内に収まっているか | `perf/cap` |

## P2 実装（54件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [GIT-IGNORE-001](phases/P2/GIT-IGNORE-001.md) | 🔴 critical | バージョン管理運用として.gitignore でシークレット除外が遵守されているか | `git/ignore` |
| [GIT-VISIBILITY-001](phases/P2/GIT-VISIBILITY-001.md) | 🔴 critical | バージョン管理運用としてリポジトリ可視性（最初は private）が遵守されているか | `git/visibility` |
| [IMPL-IDEMPOTENT-001](phases/P2/IMPL-IDEMPOTENT-001.md) | 🔴 critical | 実装として冪等性（再送・連打耐性）が遵守され、レビューと自動検査でPassしているか | `impl/idempotency` |
| [IMPL-TX-001](phases/P2/IMPL-TX-001.md) | 🔴 critical | 実装としてトランザクション境界実装が遵守され、レビューと自動検査でPassしているか | `impl/transaction` |
| [OPS-LOG-PII-001](phases/P2/OPS-LOG-PII-001.md) | 🔴 critical | 運用としてログへのPII/秘密情報混入防止が定常化し、記録・レビューが継続しているか | `ops/log` |
| [SEC-AUTH-PWD-001](phases/P2/SEC-AUTH-PWD-001.md) | 🔴 critical | 対象コード・設定においてパスワードハッシュ（argon2id推奨）に対応する防御機構が実装され、検証されているか | `security/auth` |
| [SEC-AUTHZ-ESCALATE-001](phases/P2/SEC-AUTHZ-ESCALATE-001.md) | 🔴 critical | 対象コード・設定において権限昇格防止（mass assignment）に対応する防御機構が実装され、検証されているか | `security/authz` |
| [SEC-AUTHZ-FUNC-001](phases/P2/SEC-AUTHZ-FUNC-001.md) | 🔴 critical | 対象コード・設定において機能レベル認可（admin系）に対応する防御機構が実装され、検証されているか | `security/authz` |
| [SEC-FILE-001](phases/P2/SEC-FILE-001.md) | 🔴 critical | 対象コード・設定においてファイルアップロード（拡張子/MIME/中身の三重検証）に対応する防御機構が実装され、検証されて | `security/file` |
| [SEC-IDOR-001](phases/P2/SEC-IDOR-001.md) | 🔴 critical | URLやペイロードにリソースIDを含むAPIで、リクエスト元ユーザーがそのリソースの所有者・権限保有者であることを毎回サ | `security/authz` |
| [SEC-INJECT-CMD-001](phases/P2/SEC-INJECT-CMD-001.md) | 🔴 critical | 対象コード・設定においてコマンドインジェクション防止に対応する防御機構が実装され、検証されているか | `security/injection` |
| [SEC-INJECT-SQL-001](phases/P2/SEC-INJECT-SQL-001.md) | 🔴 critical | 対象コード・設定においてパラメタライズドクエリに対応する防御機構が実装され、検証されているか | `security/injection` |
| [SEC-INPUT-001](phases/P2/SEC-INPUT-001.md) | 🔴 critical | 対象コード・設定においてサーバ側バリデーション必須（クライアントは補助）に対応する防御機構が実装され、検証されているか | `security/input` |
| [SEC-LIB-001](phases/P2/SEC-LIB-001.md) | 🔴 critical | 対象コード・設定においてライブラリ実在確認・最終更新・メンテナ活動に対応する防御機構が実装され、検証されているか | `security/supply` |
| [SEC-SECRET-001](phases/P2/SEC-SECRET-001.md) | 🔴 critical | ソースコード・設定ファイル・コミット履歴にシークレット（APIキー、トークン、パスワード）がハードコードされていないか | `security/secret` |
| [SEC-SECRET-002](phases/P2/SEC-SECRET-002.md) | 🔴 critical | 対象コード・設定において公開鍵と秘密鍵の混同回避に対応する防御機構が実装され、検証されているか | `security/secret` |
| [SEC-SSRF-001](phases/P2/SEC-SSRF-001.md) | 🔴 critical | 対象コード・設定においてSSRF対策（メタデータエンドポイント保護含む）に対応する防御機構が実装され、検証されているか | `security/ssrf` |
| [SEC-XSS-001](phases/P2/SEC-XSS-001.md) | 🔴 critical | 対象コード・設定において出力エスケープ + CSPに対応する防御機構が実装され、検証されているか | `security/xss` |
| [IMPL-DOUBLESUBMIT-001](phases/P2/IMPL-DOUBLESUBMIT-001.md) | 🟠 high | 実装として二重送信防止（front+back）が遵守され、レビューと自動検査でPassしているか | `impl/idempotency` |
| [IMPL-EXCEPT-001](phases/P2/IMPL-EXCEPT-001.md) | 🟠 high | 実装として例外の握りつぶし禁止・根本原因対処が遵守され、レビューと自動検査でPassしているか | `impl/exception` |
| [IMPL-RACE-001](phases/P2/IMPL-RACE-001.md) | 🟠 high | 実装として競合状態（楽観/悲観ロック）が遵守され、レビューと自動検査でPassしているか | `impl/concurrency` |
| [IMPL-RETRY-001](phases/P2/IMPL-RETRY-001.md) | 🟠 high | 実装としてリトライと指数バックオフが遵守され、レビューと自動検査でPassしているか | `impl/retry` |
| [IMPL-TIMEOUT-001](phases/P2/IMPL-TIMEOUT-001.md) | 🟠 high | 実装として全外部呼出にタイムアウト設定が遵守され、レビューと自動検査でPassしているか | `impl/timeout` |
| [OPS-LOG-STRUCTURE-001](phases/P2/OPS-LOG-STRUCTURE-001.md) | 🟠 high | 運用として構造化ログ・相関IDが定常化し、記録・レビューが継続しているか | `ops/log` |
| [PERF-INDEX-001](phases/P2/PERF-INDEX-001.md) | 🟠 high | 対象システムにおいてインデックス設計と効果計測が抑止され、計測値が閾値内に収まっているか | `perf/index` |
| [PERF-NPLUS1-001](phases/P2/PERF-NPLUS1-001.md) | 🟠 high | ORM経由のループ内クエリでN+1問題が発生していないか、eager loading等で抑止されているか | `perf/nplus1` |
| [SEC-AUTH-BRUTE-001](phases/P2/SEC-AUTH-BRUTE-001.md) | 🟠 high | 対象コード・設定においてブルートフォース・credential stuffing 対策に対応する防御機構が実装され、検証 | `security/auth` |
| [SEC-AUTH-CSRF-001](phases/P2/SEC-AUTH-CSRF-001.md) | 🟠 high | 対象コード・設定においてCSRF対策（SameSite/トークン）に対応する防御機構が実装され、検証されているか | `security/auth` |
| [SEC-AUTH-MFA-001](phases/P2/SEC-AUTH-MFA-001.md) | 🟠 high | 対象コード・設定において多要素認証に対応する防御機構が実装され、検証されているか | `security/auth` |
| [SEC-AUTH-SESSION-001](phases/P2/SEC-AUTH-SESSION-001.md) | 🟠 high | 対象コード・設定においてセッション管理・固定攻撃対策に対応する防御機構が実装され、検証されているか | `security/auth` |
| [SEC-AUTHZ-DECO-001](phases/P2/SEC-AUTHZ-DECO-001.md) | 🟠 high | 対象コード・設定において認可デコレータの標準化と漏れ検出に対応する防御機構が実装され、検証されているか | `security/authz` |
| [SEC-ERROR-001](phases/P2/SEC-ERROR-001.md) | 🟠 high | 対象コード・設定においてエラー情報漏洩（スタックトレース/SQL/path）に対応する防御機構が実装され、検証されている | `security/error` |
| [SEC-FILE-002](phases/P2/SEC-FILE-002.md) | 🟠 high | 対象コード・設定においてアップロードサイズ上限・ZipBomb対策に対応する防御機構が実装され、検証されているか | `security/file` |
| [SEC-FILE-003](phases/P2/SEC-FILE-003.md) | 🟠 high | 対象コード・設定においてSVG XSS・ファイル名サニタイズに対応する防御機構が実装され、検証されているか | `security/file` |
| [SEC-INJECT-XXE-001](phases/P2/SEC-INJECT-XXE-001.md) | 🟠 high | 対象コード・設定においてXML外部エンティティ防止に対応する防御機構が実装され、検証されているか | `security/injection` |
| [SEC-LIB-002](phases/P2/SEC-LIB-002.md) | 🟠 high | 対象コード・設定においてバージョン固定（lock file）とSBOMに対応する防御機構が実装され、検証されているか | `security/supply` |
| [SEC-LIB-003](phases/P2/SEC-LIB-003.md) | 🟠 high | 対象コード・設定においてタイポスクワッティング・依存数最小化に対応する防御機構が実装され、検証されているか | `security/supply` |
| [SEC-LIB-004](phases/P2/SEC-LIB-004.md) | 🟠 high | 対象コード・設定において脆弱性スキャン（Snyk/Dependabot/Trivy）に対応する防御機構が実装され、検証さ | `security/supply` |
| [SEC-PATHTRAV-001](phases/P2/SEC-PATHTRAV-001.md) | 🟠 high | 対象コード・設定においてパストラバーサル防止に対応する防御機構が実装され、検証されているか | `security/path` |
| [SEC-SECRET-003](phases/P2/SEC-SECRET-003.md) | 🟠 high | 対象コード・設定において最小権限の鍵発行・本番/テスト分離に対応する防御機構が実装され、検証されているか | `security/secret` |
| [SEC-SECRET-004](phases/P2/SEC-SECRET-004.md) | 🟠 high | 対象コード・設定においてログ/エラー/AIプロンプトへの混入防止に対応する防御機構が実装され、検証されているか | `security/secret` |
| [SEC-SECRET-005](phases/P2/SEC-SECRET-005.md) | 🟠 high | 対象コード・設定において鍵ローテーション計画と再発行手順事前確認に対応する防御機構が実装され、検証されているか | `security/secret` |
| [GIT-COMMIT-001](phases/P2/GIT-COMMIT-001.md) | 🟡 medium | バージョン管理運用として小さく頻繁なコミットが遵守されているか | `git/commit` |
| [IMPL-DEADLOCK-001](phases/P2/IMPL-DEADLOCK-001.md) | 🟡 medium | 実装としてデッドロック防止が遵守され、レビューと自動検査でPassしているか | `impl/concurrency` |
| [OPS-LOG-LEVEL-001](phases/P2/OPS-LOG-LEVEL-001.md) | 🟡 medium | 運用としてログレベル設計（運用想定）が定常化し、記録・レビューが継続しているか | `ops/log` |
| [PERF-FETCH-001](phases/P2/PERF-FETCH-001.md) | 🟡 medium | 対象システムにおいて不要カラム・行のフェッチ回避が抑止され、計測値が閾値内に収まっているか | `perf/fetch` |
| [PERF-MEMORY-001](phases/P2/PERF-MEMORY-001.md) | 🟡 medium | 対象システムにおいてメモリリーク（特に長寿命プロセス）が抑止され、計測値が閾値内に収まっているか | `perf/memory` |
| [PERF-POOL-001](phases/P2/PERF-POOL-001.md) | 🟡 medium | 対象システムにおいてコネクションプール枯渇防止が抑止され、計測値が閾値内に収まっているか | `perf/pool` |
| [QUALITY-COMMENT-001](phases/P2/QUALITY-COMMENT-001.md) | 🟡 medium | コード品質としてコメントは「なぜ」を書くが遵守され、Lint・レビューでPassしているか | `quality/comment` |
| [QUALITY-COMPLEX-001](phases/P2/QUALITY-COMPLEX-001.md) | 🟡 medium | コード品質として関数長・循環的複雑度が遵守され、Lint・レビューでPassしているか | `quality/complexity` |
| [QUALITY-DEAD-001](phases/P2/QUALITY-DEAD-001.md) | 🟡 medium | コード品質として死コード・コメントアウト除去が遵守され、Lint・レビューでPassしているか | `quality/dead` |
| [QUALITY-DRY-001](phases/P2/QUALITY-DRY-001.md) | 🟡 medium | コード品質としてDRY 原則（直し漏れ防止）が遵守され、Lint・レビューでPassしているか | `quality/dry` |
| [QUALITY-NAMING-001](phases/P2/QUALITY-NAMING-001.md) | 🟡 medium | コード品質として命名規則・自己説明的コードが遵守され、Lint・レビューでPassしているか | `quality/naming` |
| [SEC-INJECT-LDAP-001](phases/P2/SEC-INJECT-LDAP-001.md) | 🟡 medium | 対象コード・設定においてLDAP/NoSQL/Template インジェクションに対応する防御機構が実装され、検証されて | `security/injection` |

## P3 テスト（13件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [TEST-MIGRATE-001](phases/P3/TEST-MIGRATE-001.md) | 🔴 critical | テスト戦略・実装としてマイグレーションのドライランが遵守され、CIで継続実行されているか | `test/migration` |
| [TEST-AIGEN-001](phases/P3/TEST-AIGEN-001.md) | 🟠 high | テスト戦略・実装としてAI生成テストの段階指示（仕様書→洗出し→実装→結合）が遵守され、CIで継続実行されているか | `test/aigen` |
| [TEST-DAST-001](phases/P3/TEST-DAST-001.md) | 🟠 high | テスト戦略・実装として動的セキュリティ解析が遵守され、CIで継続実行されているか | `test/dast` |
| [TEST-FLAKY-001](phases/P3/TEST-FLAKY-001.md) | 🟠 high | テスト戦略・実装としてフレーキーテスト排除が遵守され、CIで継続実行されているか | `test/flaky` |
| [TEST-LLM-EVAL-001](phases/P3/TEST-LLM-EVAL-001.md) | 🟠 high | テスト戦略・実装としてLLM出力の回帰評価セットが遵守され、CIで継続実行されているか | `test/llm` |
| [TEST-LOAD-001](phases/P3/TEST-LOAD-001.md) | 🟠 high | テスト戦略・実装として想定×2の負荷試験が遵守され、CIで継続実行されているか | `test/load` |
| [TEST-MOCK-001](phases/P3/TEST-MOCK-001.md) | 🟠 high | テスト戦略・実装としてモック濫用回避が遵守され、CIで継続実行されているか | `test/mock` |
| [TEST-PYRAMID-001](phases/P3/TEST-PYRAMID-001.md) | 🟠 high | テスト戦略・実装としてテストピラミッドと責務分離が遵守され、CIで継続実行されているか | `test/strategy` |
| [TEST-SAST-001](phases/P3/TEST-SAST-001.md) | 🟠 high | テスト戦略・実装として静的セキュリティ解析が遵守され、CIで継続実行されているか | `test/sast` |
| [TEST-AAA-001](phases/P3/TEST-AAA-001.md) | 🟡 medium | テスト戦略・実装としてArrange/Act/Assert 分離が遵守され、CIで継続実行されているか | `test/structure` |
| [TEST-CHAOS-001](phases/P3/TEST-CHAOS-001.md) | 🟡 medium | テスト戦略・実装としてカオスエンジニアリングが遵守され、CIで継続実行されているか | `test/chaos` |
| [TEST-FUZZ-001](phases/P3/TEST-FUZZ-001.md) | 🟡 medium | テスト戦略・実装としてファジングが遵守され、CIで継続実行されているか | `test/fuzz` |
| [TEST-NAMING-001](phases/P3/TEST-NAMING-001.md) | 🟡 medium | テスト戦略・実装としてテスト名を仕様書として書くが遵守され、CIで継続実行されているか | `test/naming` |

## P4 ステージング・受入（8件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [STAGE-BACKUP-001](phases/P4/STAGE-BACKUP-001.md) | 🔴 critical | ステージング環境で自動バックアップ動作確認が実施され、合格判定が記録されているか | `stage/backup` |
| [STAGE-PII-MASK-001](phases/P4/STAGE-PII-MASK-001.md) | 🔴 critical | ステージング環境でステージング用PII疑似化が実施され、合格判定が記録されているか | `stage/pii` |
| [STAGE-RESTORE-001](phases/P4/STAGE-RESTORE-001.md) | 🔴 critical | 本番投入前にバックアップからのリストア演習を実施し、データ整合性とRPO/RTO目標達成を確認したか | `stage/restore` |
| [STAGE-BUDGET-001](phases/P4/STAGE-BUDGET-001.md) | 🟠 high | ステージング環境で予算アラート発火確認が実施され、合格判定が記録されているか | `stage/budget` |
| [STAGE-OBS-001](phases/P4/STAGE-OBS-001.md) | 🟠 high | ステージング環境で観測の疎通確認・アラート閾値検証が実施され、合格判定が記録されているか | `stage/obs` |
| [STAGE-PARITY-001](phases/P4/STAGE-PARITY-001.md) | 🟠 high | ステージング環境で本番相当データ・構成での再現が実施され、合格判定が記録されているか | `stage/parity` |
| [STAGE-DOC-001](phases/P4/STAGE-DOC-001.md) | 🟡 medium | ステージング環境でランブック・運用手順整備が実施され、合格判定が記録されているか | `stage/doc` |
| [STAGE-HEALTH-001](phases/P4/STAGE-HEALTH-001.md) | 🟡 medium | ステージング環境でヘルスチェックエンドポイントが実施され、合格判定が記録されているか | `stage/health` |

## P5 リリース（9件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [REL-ROLLBACK-001](phases/P5/REL-ROLLBACK-001.md) | 🔴 critical | リリース運用としてロールバック手順検証が準備・検証されているか | `release/rollback` |
| [REL-BLUEGREEN-001](phases/P5/REL-BLUEGREEN-001.md) | 🟠 high | リリース運用としてBlue-Green デプロイが準備・検証されているか | `release/bluegreen` |
| [REL-CANARY-001](phases/P5/REL-CANARY-001.md) | 🟠 high | リリース運用としてカナリアリリースが準備・検証されているか | `release/canary` |
| [REL-DISCLOSURE-001](phases/P5/REL-DISCLOSURE-001.md) | 🟠 high | リリース運用として障害公表テンプレ準備が準備・検証されているか | `release/disclosure` |
| [REL-FLAG-001](phases/P5/REL-FLAG-001.md) | 🟠 high | リリース運用としてFeature Flag・キルスイッチが準備・検証されているか | `release/flag` |
| [REL-MIGRATE-ZDT-001](phases/P5/REL-MIGRATE-ZDT-001.md) | 🟠 high | リリース運用としてゼロダウンタイム移行（expand-migrate-contract）が準備・検証されているか | `release/migrate` |
| [REL-ONCALL-001](phases/P5/REL-ONCALL-001.md) | 🟠 high | リリース運用としてオンコール体制・エスカレーションが準備・検証されているか | `release/oncall` |
| [REL-RUNBOOK-001](phases/P5/REL-RUNBOOK-001.md) | 🟠 high | リリース運用としてインシデントランブック準備が準備・検証されているか | `release/runbook` |
| [REL-COMM-001](phases/P5/REL-COMM-001.md) | 🟡 medium | リリース運用としてリリースノート・関係者通知が準備・検証されているか | `release/comm` |

## P6 運用・進化（13件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [OPS-DR-DRILL-001](phases/P6/OPS-DR-DRILL-001.md) | 🔴 critical | 運用としてDR・リストア演習（四半期）が定常化し、記録・レビューが継続しているか | `ops/dr` |
| [OPS-ACCESS-REV-001](phases/P6/OPS-ACCESS-REV-001.md) | 🟠 high | 運用としてアクセス権定期レビュー（退職者含む）が定常化し、記録・レビューが継続しているか | `ops/access` |
| [OPS-AUDIT-001](phases/P6/OPS-AUDIT-001.md) | 🟠 high | 運用として監査ログ別ストレージ保全が定常化し、記録・レビューが継続しているか | `ops/audit` |
| [OPS-COST-TREND-001](phases/P6/OPS-COST-TREND-001.md) | 🟠 high | 運用としてコストトレンド週次/月次確認が定常化し、記録・レビューが継続しているか | `ops/cost` |
| [OPS-INCIDENT-001](phases/P6/OPS-INCIDENT-001.md) | 🟠 high | 運用としてインシデント対応プロセス（止める→記録→通知→分析）が定常化し、記録・レビューが継続しているか | `ops/incident` |
| [OPS-KEY-ROTATE-001](phases/P6/OPS-KEY-ROTATE-001.md) | 🟠 high | 運用として鍵ローテーション運用が定常化し、記録・レビューが継続しているか | `ops/keys` |
| [OPS-LOG-REVIEW-001](phases/P6/OPS-LOG-REVIEW-001.md) | 🟠 high | 運用としてログ定期確認の習慣化が定常化し、記録・レビューが継続しているか | `ops/log` |
| [OPS-MON-ALERT-001](phases/P6/OPS-MON-ALERT-001.md) | 🟠 high | 運用としてアラート設計（疲れ防止）が定常化し、記録・レビューが継続しているか | `ops/alert` |
| [OPS-MON-GOLDEN-001](phases/P6/OPS-MON-GOLDEN-001.md) | 🟠 high | 運用としてゴールデンシグナル監視が定常化し、記録・レビューが継続しているか | `ops/monitoring` |
| [OPS-PATCH-001](phases/P6/OPS-PATCH-001.md) | 🟠 high | 運用として依存ライブラリ脆弱性パッチ運用が定常化し、記録・レビューが継続しているか | `ops/patch` |
| [OPS-POSTMORTEM-001](phases/P6/OPS-POSTMORTEM-001.md) | 🟠 high | 運用としてBlameless ポストモーテムが定常化し、記録・レビューが継続しているか | `ops/postmortem` |
| [OPS-RCA-001](phases/P6/OPS-RCA-001.md) | 🟠 high | 運用として根本原因分析（仕組み + 抽象化）が定常化し、記録・レビューが継続しているか | `ops/rca` |
| [OPS-CAPACITY-001](phases/P6/OPS-CAPACITY-001.md) | 🟡 medium | 運用としてキャパシティプランニングが定常化し、記録・レビューが継続しているか | `ops/capacity` |

## Cross-cutting（31件）

| ID | Severity | Title | Gate |
|---|---|---|---|
| [COMPLY-GDPR-001](cross-cutting/compliance/COMPLY-GDPR-001.md) | 🔴 critical | GDPR対応（越境含む）に必要な要件・統制が満たされ証跡が保管されているか | `compliance/gdpr` |
| [COMPLY-PCIDSS-001](cross-cutting/compliance/COMPLY-PCIDSS-001.md) | 🔴 critical | PCI-DSS（決済情報）に必要な要件・統制が満たされ証跡が保管されているか | `compliance/pcidss` |
| [COMPLY-PIPA-JP-001](cross-cutting/compliance/COMPLY-PIPA-JP-001.md) | 🔴 critical | 個人情報保護法（日本）対応に必要な要件・統制が満たされ証跡が保管されているか | `compliance/pipa` |
| [GOV-AI-DATAFLOW-001](cross-cutting/governance/GOV-AI-DATAFLOW-001.md) | 🔴 critical | 組織統制としてAIに貼り付ける情報の境界が方針化され、遵守状況が監査可能か | `governance/aidata` |
| [GOV-SHADOW-AI-001](cross-cutting/governance/GOV-SHADOW-AI-001.md) | 🔴 critical | 組織統制として会社未承認AIツール使用禁止が方針化され、遵守状況が監査可能か | `governance/shadow` |
| [LLM-AGENCY-001](cross-cutting/llm/LLM-AGENCY-001.md) | 🔴 critical | LLM/エージェント運用において過剰なエージェント権限の制限に対する防御・検証機構が機能しているか | `llm/agency` |
| [LLM-INJECT-DIR-001](cross-cutting/llm/LLM-INJECT-DIR-001.md) | 🔴 critical | LLM/エージェント運用においてプロンプトインジェクション（直接）に対する防御・検証機構が機能しているか | `llm/injection` |
| [LLM-INJECT-INDIR-001](cross-cutting/llm/LLM-INJECT-INDIR-001.md) | 🔴 critical | RAG/エージェントが取り込む外部文書・Web内容に埋め込まれた間接プロンプトインジェクションに対して、隔離・無害化・検 | `llm/injection` |
| [LLM-OUTPUT-TRUST-001](cross-cutting/llm/LLM-OUTPUT-TRUST-001.md) | 🔴 critical | LLM/エージェント運用においてLLM出力を無検証で実行しない（コード/SQL/コマンド）に対する防御・検証機構が機能し | `llm/output` |
| [LLM-PII-001](cross-cutting/llm/LLM-PII-001.md) | 🔴 critical | LLM/エージェント運用においてプロンプト内PII送信制御・学習オプトアウトに対する防御・検証機構が機能しているか | `llm/pii` |
| [LLM-TOOL-POISON-001](cross-cutting/llm/LLM-TOOL-POISON-001.md) | 🔴 critical | LLM/エージェント運用においてツール汚染（MCP含む）に対する防御・検証機構が機能しているか | `llm/tool` |
| [COMPLY-HIPAA-001](cross-cutting/compliance/COMPLY-HIPAA-001.md) | 🟠 high | HIPAA（医療情報）に必要な要件・統制が満たされ証跡が保管されているか | `compliance/hipaa` |
| [COMPLY-RETAIN-001](cross-cutting/compliance/COMPLY-RETAIN-001.md) | 🟠 high | データ保持期間ポリシーに必要な要件・統制が満たされ証跡が保管されているか | `compliance/retain` |
| [COMPLY-SOC2-001](cross-cutting/compliance/COMPLY-SOC2-001.md) | 🟠 high | SOC 2に必要な要件・統制が満たされ証跡が保管されているか | `compliance/soc2` |
| [DOC-RUNBOOK-001](cross-cutting/doc/DOC-RUNBOOK-001.md) | 🟠 high | ドキュメントとしてランブック整備が整備され、最新性が維持されているか | `doc/runbook` |
| [GOV-AUDIT-TRAIL-001](cross-cutting/governance/GOV-AUDIT-TRAIL-001.md) | 🟠 high | 組織統制として操作監査証跡が方針化され、遵守状況が監査可能か | `governance/audit` |
| [GOV-INTERNAL-SYSTEM-001](cross-cutting/governance/GOV-INTERNAL-SYSTEM-001.md) | 🟠 high | 組織統制として「社内だから安全」幻想の排除（横移動防止）が方針化され、遵守状況が監査可能か | `governance/internal` |
| [LLM-COST-RUNAWAY-001](cross-cutting/llm/LLM-COST-RUNAWAY-001.md) | 🟠 high | LLM/エージェント運用においてLLM呼出コスト暴走防止（ハードリミット）に対する防御・検証機構が機能しているか | `llm/cost` |
| [LLM-DRIFT-001](cross-cutting/llm/LLM-DRIFT-001.md) | 🟠 high | LLM/エージェント運用においてモデル更新時の品質ドリフト検知に対する防御・検証機構が機能しているか | `llm/drift` |
| [LLM-EVAL-REGR-001](cross-cutting/llm/LLM-EVAL-REGR-001.md) | 🟠 high | LLM/エージェント運用において評価セットによる回帰検知に対する防御・検証機構が機能しているか | `llm/eval` |
| [LLM-HALLUCIN-001](cross-cutting/llm/LLM-HALLUCIN-001.md) | 🟠 high | LLM/エージェント運用においてハルシネーション検知・引用必須化に対する防御・検証機構が機能しているか | `llm/hallucination` |
| [LLM-LIB-HALLUCIN-001](cross-cutting/llm/LLM-LIB-HALLUCIN-001.md) | 🟠 high | LLM/エージェント運用においてAIが提案する架空ライブラリの検証に対する防御・検証機構が機能しているか | `llm/lib` |
| [UX-A11Y-WCAG-001](cross-cutting/ux/UX-A11Y-WCAG-001.md) | 🟠 high | UI/UXとしてWCAG/WAI-ARIA 適合が満たされ、自動・手動テストで確認されているか | `ux/a11y` |
| [DOC-ADR-001](cross-cutting/doc/DOC-ADR-001.md) | 🟡 medium | ドキュメントとしてADR（Architecture Decision Record）が整備され、最新性が維持されているか | `doc/adr` |
| [DOC-API-001](cross-cutting/doc/DOC-API-001.md) | 🟡 medium | ドキュメントとしてAPI ドキュメント（OpenAPI等）が整備され、最新性が維持されているか | `doc/api` |
| [DOC-DIAGRAM-001](cross-cutting/doc/DOC-DIAGRAM-001.md) | 🟡 medium | ドキュメントとして図（C4等）保守が整備され、最新性が維持されているか | `doc/diagram` |
| [DOC-README-001](cross-cutting/doc/DOC-README-001.md) | 🟡 medium | ドキュメントとしてREADME 必須項目が整備され、最新性が維持されているか | `doc/readme` |
| [UX-A11Y-COLOR-001](cross-cutting/ux/UX-A11Y-COLOR-001.md) | 🟡 medium | UI/UXとして色覚多様性対応が満たされ、自動・手動テストで確認されているか | `ux/a11y` |
| [UX-A11Y-KEYBOARD-001](cross-cutting/ux/UX-A11Y-KEYBOARD-001.md) | 🟡 medium | UI/UXとしてキーボード操作完全性が満たされ、自動・手動テストで確認されているか | `ux/a11y` |
| [UX-EMPTYSTATE-001](cross-cutting/ux/UX-EMPTYSTATE-001.md) | 🟡 medium | UI/UXとして空状態・エラー状態のUXが満たされ、自動・手動テストで確認されているか | `ux/state` |
| [UX-FEEDBACK-001](cross-cutting/ux/UX-FEEDBACK-001.md) | 🟡 medium | UI/UXとしてフィードバック設計（即時性）が満たされ、自動・手動テストで確認されているか | `ux/feedback` |

