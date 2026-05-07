---
name: LLM出力検証_許可リスト
description: LLM が生成した SQL / コマンド / コード / API 呼び出しを Sandbox 実行し、許可リスト方式で検証するスキル。DDL/DML 完全禁止、AST パース、Read-only 強制。LLM-OUTPUT-TRUST-001 / OWASP LLM05 (Improper Output Handling) 対応。
---

# LLM 出力検証 / 許可リスト

## 対応する品質評価ID

- **LLM-OUTPUT-TRUST-001** (Fail → Pass): LLM 出力 (SQL/コード/コマンド) の信頼検証が未整備
- **OWASP LLM05** (Improper Output Handling): 出力を信頼して下流処理に流す脅威
- **OWASP LLM06** (Excessive Agency): 出力経由で過剰な操作を実行
- **GOV-AI-DATAFLOW-001** (Conditional → Pass): AI データフロー出口の信頼ガバナンス

## 概要

LLM の出力を**そのまま下流処理 (DB / シェル / API) に流すと、危険な操作が実行される**可能性がある。Text2SQL、コードアシスト、Tool-calling Agent では特に深刻。

防御の基本は以下:

1. **Sandbox 実行** : 出力を本番ではなく隔離環境で実行
2. **許可リスト方式** (allowlist) : 通る操作を限定列挙、それ以外は全 deny
3. **AST/構文パース** : 文字列マッチではなく構文木で判定
4. **DDL/DML 完全禁止** : 破壊的操作はエージェント権限から物理的に剥奪
5. **二重検証** : LLM-judge による自然言語と SQL の意図一致チェック

## いつ使うか

- LLM が SQL を生成して DB に発行する全ユースケース (UC-011 Text2SQL)
- LLM がシェルコマンドを生成して OS で実行
- LLM がコードを生成し eval/exec する場合
- LLM が外部 API 呼び出しの引数を生成
- LLM の Tool-calling 機能を本番に乗せる場合

## 手順

### ステップ 1: 操作カテゴリの分類と許可方針

各操作を 3 段階で分類:

| カテゴリ | 例 | 許可方針 |
|---|---|---|
| 安全 (allowlist 通過なら可) | SELECT, EXPLAIN, SHOW | 許可リスト + Sandbox |
| 危険 (一切禁止) | DROP, TRUNCATE, ALTER, GRANT | 完全禁止 (AST 段階で reject) |
| 条件付 (Human承認必須) | INSERT (監査用)、UPDATE (限定範囲) | 承認フロー連動 |

### ステップ 2: SQL 出力の検証 (UC-011 Text2SQL)

#### 2-1. 許可リスト構成

```yaml
sql_allowlist:
  allowed_statements: [SELECT, WITH, EXPLAIN]
  forbidden_statements:
    - INSERT
    - UPDATE
    - DELETE
    - MERGE
    - CREATE
    - DROP
    - ALTER
    - TRUNCATE
    - GRANT
    - REVOKE
    - COPY     # Snowflake stage への書き込み
    - PUT      # ファイルアップロード
    - REMOVE   # ファイル削除
  allowed_schemas:
    - analytics_gold
    - analytics_semantic
  forbidden_schemas:
    - bronze.*       # 生 PII 含む
    - silver.pii_*
    - infrastructure.*
  allowed_functions:
    - COUNT, SUM, AVG, MIN, MAX, MEDIAN
    - GROUP BY, ORDER BY, JOIN
    - CASE WHEN, COALESCE
  forbidden_functions:
    - SYSTEM$*       # Snowflake 管理関数
    - GET_DDL        # DDL 抽出
    - CURRENT_USER   # ユーザー情報露出
  max_join_depth: 5
  max_query_time_seconds: 60
  max_rows_returned: 10000
```

#### 2-2. AST パースで判定

文字列の `LIKE '%DROP%'` ではなく**SQL 構文木**で判定する。`sqlglot` 等を使う。

```python
import sqlglot
from sqlglot import expressions as exp

def validate_sql(sql: str, allowlist: dict) -> ValidationResult:
    issues = []
    try:
        parsed = sqlglot.parse_one(sql, dialect="snowflake")
    except Exception as e:
        return ValidationResult(valid=False, issues=[f"Parse error: {e}"])

    # 1. statement type check
    statement_type = type(parsed).__name__
    if statement_type not in allowlist["allowed_statements"]:
        issues.append(f"FORBIDDEN_STATEMENT: {statement_type}")

    # 2. table reference check
    for table in parsed.find_all(exp.Table):
        full_name = f"{table.db}.{table.name}".lower()
        if any(full_name.startswith(prefix.replace('*', '').lower())
               for prefix in allowlist["forbidden_schemas"]):
            issues.append(f"FORBIDDEN_SCHEMA: {full_name}")

    # 3. function call check
    for func in parsed.find_all(exp.Func):
        name = func.this.upper() if hasattr(func.this, "upper") else str(func.this).upper()
        if name in allowlist["forbidden_functions"]:
            issues.append(f"FORBIDDEN_FUNCTION: {name}")

    # 4. JOIN depth check
    join_depth = len(list(parsed.find_all(exp.Join)))
    if join_depth > allowlist["max_join_depth"]:
        issues.append(f"JOIN_TOO_DEEP: {join_depth}")

    return ValidationResult(valid=len(issues) == 0, issues=issues)
```

#### 2-3. Snowflake ROLE による物理的制限 (二重防御)

許可リストだけでなく、**実行ロール自体に DML/DDL 権限を持たせない**。

```sql
-- Snowflake で AGT-003 の実行ロール作成
CREATE ROLE AGENT_TEXT2SQL_ROLE;

-- SELECT のみ付与
GRANT USAGE ON DATABASE ANALYTICS TO ROLE AGENT_TEXT2SQL_ROLE;
GRANT USAGE ON SCHEMA ANALYTICS.GOLD TO ROLE AGENT_TEXT2SQL_ROLE;
GRANT SELECT ON ALL TABLES IN SCHEMA ANALYTICS.GOLD TO ROLE AGENT_TEXT2SQL_ROLE;

-- DML/DDL 権限は一切付与しない
-- INSERT/UPDATE/DELETE/CREATE/DROP/ALTER は GRANT しない

-- Bronze/Silver は明示的に DENY (Future Grants で覆われないため)
REVOKE ALL ON DATABASE BRONZE FROM ROLE AGENT_TEXT2SQL_ROLE;
```

これにより、**LLM が DROP TABLE を生成しても実行段階で権限不足エラー**となり安全。

### ステップ 3: シェルコマンドの検証

LLM が `rm -rf /tmp/x` のようなコマンドを生成する場合。

```yaml
shell_allowlist:
  allowed_commands:
    - ls
    - cat
    - grep
    - awk
    - sed
    - head
    - tail
    - wc
    - sort
    - uniq
  forbidden_commands:
    - rm
    - mv
    - cp -r       # 再帰コピーは要審査
    - chmod
    - chown
    - sudo
    - kill
    - shutdown
    - reboot
    - dd
    - mkfs
    - curl        # 外部送信防止
    - wget
    - nc          # netcat
    - ssh
    - scp
  forbidden_patterns:
    - "> /etc/*"
    - ">> /etc/*"
    - "&& "       # コマンドチェーン (最低限の禁止例)
    - "; "
    - "| sh"
    - "| bash"
  sandbox: docker_container_readonly_fs
```

```python
import shlex

def validate_shell(command: str, allowlist: dict) -> ValidationResult:
    tokens = shlex.split(command)
    base_cmd = tokens[0] if tokens else ""

    if base_cmd not in allowlist["allowed_commands"]:
        return ValidationResult(valid=False, issues=[f"FORBIDDEN_COMMAND: {base_cmd}"])

    for pattern in allowlist["forbidden_patterns"]:
        if pattern in command:
            return ValidationResult(valid=False, issues=[f"FORBIDDEN_PATTERN: {pattern}"])

    return ValidationResult(valid=True)
```

### ステップ 4: コードの Sandbox 実行

LLM 生成コードを `eval`/`exec` するのではなく、**隔離コンテナ**で実行。

| Sandbox | 用途 | 例 |
|---|---|---|
| Docker (--read-only --network=none) | 一時的なコード検証 | sqlglot 検証、データ可視化 |
| gVisor | システムコール制限 | より高セキュリティ |
| Firecracker / E2B | サブセカンド起動 | OpenAI Code Interpreter 同等 |
| AWS Lambda / Azure Functions (限定 IAM) | サーバーレス Sandbox | 本番統合用 |

```python
import docker

def execute_in_sandbox(code: str, language: str = "python") -> ExecutionResult:
    client = docker.from_env()
    container = client.containers.run(
        image=f"sandbox-{language}:latest",
        command=["python", "-c", code],
        network_mode="none",
        read_only=True,
        mem_limit="256m",
        cpu_quota=50000,        # 50% CPU
        pids_limit=64,
        detach=True,
        remove=True,
        stdin_open=False,
    )
    try:
        container.wait(timeout=30)
        logs = container.logs().decode("utf-8")
        return ExecutionResult(success=True, output=logs)
    except docker.errors.ContainerError as e:
        return ExecutionResult(success=False, error=str(e))
    finally:
        container.kill() if container.status == "running" else None
```

### ステップ 5: 二重検証 (LLM-judge による意図一致)

ユーザーの自然言語クエリと生成 SQL の**意図が一致**しているかを別 LLM で判定。

```yaml
llm_judge:
  task: "ユーザーの質問と生成SQLが同じ意図か判定"
  prompt: |
    ユーザー質問: "{user_query}"
    生成 SQL: "{generated_sql}"

    以下を JSON で返答:
    {
      "intent_match": true/false,
      "confidence": 0.0-1.0,
      "reason": "..."
    }
  threshold: 0.85  # confidence < 0.85 は実行禁止
```

これにより、プロンプトインジェクションで「SELECT 風だが実は SELECT * FROM users WHERE 1=1 OR 1=1」のような変質した SQL を検出。

### ステップ 6: 実行ログと監査

```yaml
execution_audit:
  - timestamp: ISO8601
  - agent_id: string
  - user_query: string  # マスキング後
  - generated_output: string  # SQL/コマンド/コード
  - validation_result: enum [PASS, FAIL]
  - validation_issues: [string]
  - sandbox_executed: bool
  - rows_returned: int
  - execution_time_ms: int
  - error: string
  - human_approval: bool

monitoring:
  - metric: validation_fail_rate (1h)
    alert_threshold: 5%   # 5% 超は LLM プロンプト見直し
  - metric: forbidden_attempt_count (1h)
    alert_threshold: 10   # 10 件超でセキュリティ調査
```

## 例示: UC-011 Text2SQL

### 攻撃シナリオ

業務ユーザーが「先月の売上トップ 10 顧客を出して」と入力。LLM がプロンプトインジェクションを受けて以下を生成:

```sql
SELECT * FROM customers; DROP TABLE customers; --
```

### 防御フロー

```
1. AST パース → 2 つの statement (SELECT + DROP)
2. 許可リストチェック → DROP は forbidden_statements に該当 → REJECT
3. ロール権限 → AGENT_TEXT2SQL_ROLE には DROP 権限なし → 二重防御
4. 監査ログに forbidden_attempt として記録
5. ユーザーへ "ご質問を別の表現で頂けますか" と返答
```

### 正常フロー

```
ユーザー: "先月の売上トップ 10 顧客を出して"
LLM 出力:
  SELECT customer_id, customer_name, SUM(sales_amount) AS total
  FROM analytics_gold.fact_sales
  WHERE sale_date BETWEEN DATE_TRUNC('month', DATEADD('month', -1, CURRENT_DATE())) AND CURRENT_DATE()
  GROUP BY 1, 2
  ORDER BY total DESC
  LIMIT 10

検証:
  - 文 SELECT → OK
  - スキーマ analytics_gold → OK
  - 関数 SUM, DATE_TRUNC, DATEADD → OK
  - JOIN 数 0 → OK
  - LIMIT あり → OK
LLM-judge:
  - intent_match: true
  - confidence: 0.96
ROLE 実行:
  - 鍵: AGENT_TEXT2SQL_ROLE
  - 結果: 10 行返却

監査ログ記録 → ユーザー画面に表示
```

## 例示: UC-003 ボイスボット (チケット起票)

LLM が「チケット起票」を tool-calling で呼ぶケース。

```yaml
tool_definition:
  name: create_inquiry_ticket
  schema:
    customer_id: string  # 既存 ID のみ
    inquiry_type: enum [billing, technical, general]
    summary: string (max 500 chars)
    severity: enum [low, medium, high]
allowlist_check:
  - customer_id pattern: ^[A-Z0-9]{8}$
  - inquiry_type ∈ enum
  - summary: PII redaction 済 + 500 字以内
  - severity ∈ enum
forbidden_overrides:
  - status は受付/エスカレーションのみ (close は不可)
  - assignee は固定 (LLM 指定不可)
```

## 出力フォーマット

| 成果物 | 形式 | 場所 |
|---|---|---|
| 許可リスト定義 | YAML | `allowlists/{tool}/*.yaml` |
| 検証コード | Python | `src/validators/{sql,shell,code}_validator.py` |
| Sandbox 設定 | Dockerfile + IAM | `sandbox/Dockerfile` + `iam/sandbox-role.json` |
| Snowflake ROLE 定義 | SQL | `db/roles/agent_*.sql` |
| 監査ログスキーマ | JSON Schema | `schemas/execution-audit.json` |

## 検証方法

- [ ] 全 LLM 出力経路に許可リスト検証が入っているか
- [ ] AST パースで判定しているか (文字列マッチではない)
- [ ] DML/DDL が **物理的に** 不可能か (ROLE 権限なし)
- [ ] Sandbox の network=none / read-only fs / 30s タイムアウトが設定済か
- [ ] LLM-judge による意図一致チェックがあるか
- [ ] 実行ログが監査用に保管されているか
- [ ] 危険パターン検知時のアラートとエスカレーションがあるか
- [ ] 「許可リスト」 vs 「拒否リスト」では**許可リスト方式**を採用しているか

## pm-blueprint 連携

| 連携先 | 関係 |
|---|---|
| `custom/AI駆動開発リスク.md` R-AI-12 | プロンプトインジェクションで生成された不正出力を本書で阻止 |
| `layer-8-llm-governance/エージェント権限境界書.md` | 許可リストと ROLE 権限を一致させる |
| `layer-8-llm-governance/プロンプトインジェクション評価.md` | 攻撃成功時の被害局限策として本書が機能 |
| `layer-8-llm-governance/PII境界_DLP.md` | 出力に PII が含まれた場合の二重検出 |
| `layer-5-risk/脅威モデリング.md` STRIDE-T (改ざん) | LLM 出力の改ざんを本書の AST 検証で検知 |
| `layer-5-risk/脅威モデリング.md` STRIDE-E (権限昇格) | 物理 ROLE 制限が緩和策 |

## 参考

- OWASP, "OWASP Top 10 for LLM Applications 2025" LLM05/06
- sqlglot Documentation, "SQL Parser and Transpiler"
- E2B, "Code Interpreter Sandbox"
- AWS Lambda, "Code Execution Sandbox Best Practices"
- Snowflake, "Role-Based Access Control (RBAC)"
- gVisor Documentation, "Application Kernel for Containers"
- LangChain, "Tool Use Validation Patterns"
- OpenAI, "Function Calling Best Practices"
- Anthropic, "Claude Tool Use - Output Validation Patterns"
