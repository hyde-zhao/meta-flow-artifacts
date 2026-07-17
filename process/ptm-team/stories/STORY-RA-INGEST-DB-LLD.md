---
story_id: "STORY-RA-INGEST-DB"
canonical_story_id: "ST-RA-INGEST-DB"
title: "SQLite 数据库 schema 与 DAO 层"
source_cr: "CR-030"
feature: "FEAT-RA-INGESTION"
lld_policy: "full-lld"
tier: "Standard"
design_evidence_type: "full-lld"
created_by: "meta-dev"
created_at: "2026-07-16"
open_items: []
shared_fragments: []
---

# ST-RA-INGEST-DB: SQLite 数据库 schema 与 DAO 层 — LLD

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | SQLite 单写、受限存储和 `full-lld` 约束 |
| Story `ST-RA-INGEST-DB` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 的 SQLite Data Owner 边界。本 LLD 覆盖全部 6 张表（ticket, ticket_version, ingestion_batch, change_history, analysis_run, measure_link）的 DDL 与 DAO。
>
> **CP5 Round 2 整改**：采用方案 A — F-020 是 `data/dao.py` 的唯一写入者，一次性定义全部 6 张表的完整 DAO 接口（含 analysis_run 和 measure_link 的读写方法）。
> F-021（reverse-analysis）和 F-023（improvement-tracker）只通过 DAO 公共接口调用，不修改 `data/dao.py`。
> 此决策已由 host-orchestrator 在 QUESTION-LEDGER LCQ-ST-RA-INGEST-DB-01 中确认（选 A，2026-07-16），并在 CP5 Round 2 中从"下游追加"修正为"F-020 单写全量 DAO"。

## 1. 文件影响范围

| 文件 | 操作 | 所有权 | 说明 |
|---|---|---|---|
| `data/schema.sql` | 新建 | F-020 独占写入 | 完整 DDL（6 张表：ticket、ticket_version、ingestion_batch、change_history、analysis_run、measure_link + 索引） |
| `data/dao.py` | 新建 | **F-020 独占写入** | DAO 层。F-020 一次性定义全部 6 张表的完整 CRUD 接口（含 analysis_run 和 measure_link 的读写方法）。下游 F-021/022/023 仅调用获授权公共接口，不追加、不修改此文件。 |
| `data/.gitignore` | 新建 | F-020 独占写入 | 排除 `*.db`、`*.db-journal`、`*.db-wal`、`*.db-shm` 和 `snapshots/`；后续 Story 不修改此文件 |

**不修改的文件**: `docs/design/HLD.md`、`docs/features/feat-ra-ingestion/DESIGN.md`

## 2. 接口设计

### 2.1 DAO 公共接口

```python
# —— 连接管理 ——
def get_connection(db_path: str = "data/ptm-tse.db") -> sqlite3.Connection
# 返回: 启用 WAL 模式、外键约束的 sqlite3.Connection

# —— Ticket CRUD ——
def insert_ticket(conn, ticket: dict) -> int
# 输入: {source_ticket_id, product, module, priority, severity, status, title, description,
#         root_cause, test_missed_analysis, test_missed_phase, improvement_measures,
#         openeddate, resolveddate, raw_json, quality_flag, first_seen_at,
#         last_seen_at, source_updated_at}
# 输出: 新增行的 id
# 异常: sqlite3.IntegrityError（source_ticket_id 重复）

def upsert_ticket(conn, ticket: dict, field_diffs: dict) -> int
# 输入: 同 insert_ticket + field_diffs（{field: {old, new}}）
# 行为: source_ticket_id 已存在时 UPDATE 变更字段 + last_seen_at；不存在时 INSERT
# 返回: ticket.id（新建或已有）
# 输出副作用: 返回的 ticket_id 用于 ticket_version 写入

def get_ticket_by_source_id(conn, source_ticket_id: str) -> dict | None
# 返回: ticket 行 dict，不存在返回 None

def get_tickets_by_product(conn, product: str, limit: int = 1000) -> list[dict]

def get_tickets_by_time_range(conn, start: str, end: str) -> list[dict]

def get_tickets_by_batch(conn, batch_ref: str) -> list[dict]
# 通过 ticket_version.batch_ref 关联读取该批次的清洗记录；只返回 quality_flag='clean' 的可分析记录

# —— Ticket Version ——
def insert_ticket_version(conn, ticket_id: int, version: int,
                          previous_status: str | None, field_diffs: str,
                          batch_ref: str) -> int
# 输入: field_diffs 为 JSON string（{field: {old, new}}）
# 输出: 新行 id
# 异常: sqlite3.IntegrityError（UNIQUE(ticket_id, version) 冲突）

def get_ticket_versions(conn, ticket_id: int) -> list[dict]
# 返回: 按 version ASC 排列的历史版本

def get_next_version(conn, ticket_id: int) -> int
# 返回: 当前最大 version + 1，无历史则返回 1

# —— Ingestion Batch ——
def insert_ingestion_batch(conn, batch: dict) -> int
# 输入: {batch_id, request_url, request_params, http_status, response_hash,
#         snapshot_ref, schema_version, total_fetched, total_cleaned,
#         total_failed, quality_report_ref}
# 输出: 新行 id
# 异常: sqlite3.IntegrityError（batch_id 重复）

def get_batch(conn, batch_id: str) -> dict | None

def get_batches_by_time(conn, start: str, end: str) -> list[dict]

# —— Change History ——
def insert_change_history(conn, ticket_id: int | None, batch_ref: str,
                          change_type: str, affected_fields: str | None,
                          resolution: str, conflict_ref: str | None = None) -> int
# 输入: change_type ∈ {'new', 'modified', 'unchanged', 'conflict'}
#       resolution ∈ {'auto_merged', 'manual_queue', 'rejected'}
#       affected_fields 为 JSON array string；无稳定 ID 的 conflict 允许 ticket_id=None，
#       此时 conflict_ref 必须指向受控 conflict queue 条目

def get_changes_by_ticket(conn, ticket_id: int) -> list[dict]

def get_changes_by_batch(conn, batch_ref: str) -> list[dict]

# —— Analysis Run（F-021 消费，F-020 提供）——
def insert_analysis_run(conn, run: dict) -> int
# 输入: {run_id, batch_ref, schema_version, mapping_version, rule_version,
#         comparison_batch_ref, time_window_start, time_window_end, recompute_mode,
#         full_recompute_reason, report_refs, metric_versions}
# 输出: 新行 id
# 异常: sqlite3.IntegrityError（run_id 重复）

def get_analysis_run(conn, run_id: str) -> dict | None

def get_runs_by_batch(conn, batch_ref: str) -> list[dict]

def update_analysis_run_draft(conn, run_id: str, status: str, report_refs: str) -> bool
# 行为: ANALYSIS 仅可在 created/in_progress/completed/failed 间更新草案执行状态；不得发布
# 返回: True if updated, False if not found

def reviewer_publish_analysis_run(conn, run_id: str, reviewer_ref: str) -> bool
# 行为: 仅 reviewer 显式确认后将 completed 草案标记为 published，并写入 reviewer_ref/published_at

# —— Measure Link（F-023 消费，F-020 提供）——
def insert_measure_link(conn, measure: dict) -> int
# 输入: {proposal_ref, baseline_id, baseline_version, scope, approval_ref,
#         implementation_evidence, effectiveness_evidence,
#         observation_window_start, observation_window_end,
#         status, proposed_status, refresh_hint}
# 输出: 新行 id

def update_measure_refresh_hint(conn, proposal_ref: str, updates: dict) -> int
# 行为: 仅更新已有 proposal 的系统提示字段；不存在则拒绝，系统不得自动创建 baseline
# 返回: measure_link.id
# updates 允许的键: proposed_status, refresh_hint, refreshed_at, refreshed_by, refresh_reason
# **禁止**: status, approval_ref, baseline_id 不可通过本方法修改

def reviewer_update_measure_status(conn, proposal_ref: str, status: str,
                                   reviewer_ref: str) -> int
# 行为: reviewer 唯一可变更正式 status；记录 reviewer_ref 和更新时间

def get_measure_link(conn, proposal_ref: str) -> dict | None

def get_measures_by_status(conn, status: str) -> list[dict]

# —— 事务管理 ——
def begin_transaction(conn) -> None
def commit(conn) -> None
def rollback(conn) -> None
```

### 2.2 下游调用接口（供 F-021/022/023 消费）

下游 Feature 通过导入 `data/dao.py` 的公共函数来操作数据库。**所有写操作（INSERT/UPDATE/DELETE）均通过 F-020 提供的 DAO 接口执行；下游 Feature 不得直接执行 SQL 或追加 DAO 方法。**

F-021 消费：`get_connection()`, `get_tickets_by_product()`, `get_tickets_by_time_range()`, `get_tickets_by_batch()`, `get_ticket_versions()`, `get_batch()`, `insert_analysis_run()`, `get_analysis_run()`, `get_runs_by_batch()`, `update_analysis_run_draft()`；仅 reviewer 调用 `reviewer_publish_analysis_run()`。

F-022 消费：`get_connection()`, `get_tickets_by_product()`（只读查询分析结果）

F-023 消费：`get_connection()`, `insert_measure_link()`（仅 reviewer 创建基线）、`update_measure_refresh_hint()`（系统仅写提示）、`reviewer_update_measure_status()`（仅 reviewer）、`get_measure_link()`, `get_measures_by_status()`。

### 2.3 输入校验

| 字段 | 校验规则 | 失败行为 |
|---|---|---|
| `source_ticket_id` | 非空字符串，长度 ≤ 256 | 抛出 ValueError |
| `product` | 非空字符串 | 抛出 ValueError |
| `quality_flag` | 枚举: clean / incomplete / anomaly / blocked | 抛出 ValueError |
| `change_type` | 枚举: new / modified / unchanged / conflict | 抛出 ValueError |
| `resolution` | 枚举: auto_merged / manual_queue / rejected | 抛出 ValueError |
| `field_diffs` | 合法 JSON string | 抛出 json.JSONDecodeError |

## 3. 数据模型

### 3.1 表结构（DDL）

```sql
-- 问题单主表
CREATE TABLE ticket (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    source_ticket_id TEXT NOT NULL UNIQUE,
    product TEXT NOT NULL,
    module TEXT,
    priority TEXT,
    severity TEXT,
    status TEXT,
    title TEXT,
    description TEXT,
    root_cause TEXT,
    test_missed_analysis TEXT,
    test_missed_phase TEXT,
    improvement_measures TEXT,
    openeddate TEXT,
    resolveddate TEXT,
    raw_json TEXT NOT NULL,
    quality_flag TEXT NOT NULL DEFAULT 'clean',
    first_seen_at TEXT NOT NULL,
    last_seen_at TEXT NOT NULL,
    source_updated_at TEXT NOT NULL,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- 版本历史表
CREATE TABLE ticket_version (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ticket_id INTEGER NOT NULL REFERENCES ticket(id),
    version INTEGER NOT NULL,
    previous_status TEXT,
    field_diffs TEXT NOT NULL,
    batch_ref TEXT NOT NULL,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    UNIQUE(ticket_id, version)
);

-- 摄取批次表
CREATE TABLE ingestion_batch (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    batch_id TEXT NOT NULL UNIQUE,
    request_url TEXT NOT NULL,
    request_params TEXT NOT NULL,
    http_status INTEGER,
    response_hash TEXT,
    snapshot_ref TEXT NOT NULL,
    schema_version TEXT NOT NULL,
    total_fetched INTEGER,
    total_cleaned INTEGER,
    total_failed INTEGER,
    quality_report_ref TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- 变更历史表
CREATE TABLE change_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ticket_id INTEGER REFERENCES ticket(id),
    batch_ref TEXT NOT NULL,
    change_type TEXT NOT NULL,
    affected_fields TEXT,
    resolution TEXT,
    conflict_ref TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    CHECK (ticket_id IS NOT NULL OR conflict_ref IS NOT NULL)
);

-- 分析运行表（逻辑领域 owner: F-021；物理 DDL/DAO 写入: F-020）
CREATE TABLE analysis_run (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    run_id TEXT NOT NULL UNIQUE,
    batch_ref TEXT NOT NULL,
    comparison_batch_ref TEXT,
    schema_version TEXT NOT NULL,
    mapping_version TEXT NOT NULL,
    rule_version TEXT NOT NULL,
    time_window_start TEXT,
    time_window_end TEXT,
    recompute_mode TEXT NOT NULL DEFAULT 'full',
    full_recompute_reason TEXT,
    report_refs TEXT,
    metric_versions TEXT,
    status TEXT NOT NULL DEFAULT 'created' CHECK (status IN ('created', 'in_progress', 'completed', 'failed', 'published')),
    published_by TEXT,
    published_at TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- 措施关联表（逻辑领域 owner: F-023；物理 DDL/DAO 写入: F-020）
CREATE TABLE measure_link (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    proposal_ref TEXT NOT NULL,
    baseline_id TEXT,
    baseline_version INTEGER DEFAULT 1,
    scope TEXT,
    approval_ref TEXT,
    implementation_evidence TEXT,
    effectiveness_evidence TEXT,
    observation_window_start TEXT,
    observation_window_end TEXT,
    status TEXT NOT NULL DEFAULT 'needs-baseline',
    proposed_status TEXT CHECK (proposed_status IS NULL OR proposed_status IN ('needs-baseline', 'active', 'completed', 'needs-review', 'superseded')),
    refresh_hint TEXT,
    refreshed_at TEXT,
    refreshed_by TEXT,
    refresh_reason TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
```

### 3.2 索引

```sql
CREATE INDEX idx_ticket_product ON ticket(product);
CREATE INDEX idx_ticket_severity ON ticket(severity);
CREATE INDEX idx_ticket_quality_flag ON ticket(quality_flag);
CREATE INDEX idx_ticket_first_seen ON ticket(first_seen_at);
CREATE INDEX idx_ticket_last_seen ON ticket(last_seen_at);
CREATE INDEX idx_ticket_version_ticket ON ticket_version(ticket_id);
CREATE INDEX idx_ingestion_batch_created ON ingestion_batch(created_at);
CREATE INDEX idx_change_history_ticket ON change_history(ticket_id);
CREATE INDEX idx_change_history_batch ON change_history(batch_ref);
CREATE INDEX idx_analysis_run_batch ON analysis_run(batch_ref);
CREATE INDEX idx_analysis_run_status ON analysis_run(status);
CREATE INDEX idx_measure_link_proposal ON measure_link(proposal_ref);
CREATE INDEX idx_measure_link_status ON measure_link(status);
```

### 3.3 约束清单

| 约束 | 表 | 类型 | 说明 |
|---|---|---|---|
| `source_ticket_id UNIQUE` | ticket | UNIQUE | 防止重复摄取同源问题单 |
| `product NOT NULL` | ticket | NOT NULL | 强制产品归属 |
| `raw_json NOT NULL` | ticket | NOT NULL | 原始响应不可为空 |
| `quality_flag NOT NULL DEFAULT 'clean'` | ticket | NOT NULL | 每条记录必须有质量标记 |
| `UNIQUE(ticket_id, version)` | ticket_version | UNIQUE | 每版本号唯一 |
| `ticket_id REFERENCES ticket(id)` | ticket_version | FK | 版本必须关联有效 ticket |
| `ticket_id REFERENCES ticket(id)` | change_history | FK | 变更必须关联有效 ticket |
| `batch_id UNIQUE` | ingestion_batch | UNIQUE | 批次 ID 唯一 |
| `run_id UNIQUE` | analysis_run | UNIQUE | 分析运行 ID 唯一 |
| `batch_ref NOT NULL` | analysis_run | NOT NULL | 分析运行必须绑定批次 |
| `recompute_mode IN ('full','incremental')` | analysis_run | CHECK | 仅允许 full 或 incremental |
| `proposal_ref NOT NULL` | measure_link | NOT NULL | 必须关联 CA/PA Proposal |
| `status IN ('needs-baseline','active','completed','needs-review','superseded')` | measure_link | CHECK | 仅允许合法基线状态 |
| `proposed_status` 枚举或 NULL | measure_link | CHECK | 系统只能写刷新建议，不得替代正式状态 |
| `ticket_id IS NULL` 时 `conflict_ref NOT NULL` | change_history | DAO 校验 | 无稳定 ID 的冲突必须可追溯到受控队列条目 |

### 3.4 字段语义

| 字段 | 含义 | 写入时机 | 源数据 |
|---|---|---|---|
| `first_seen_at` | 首次摄取的 batch.created_at | INSERT（首次） | 采集时间戳 |
| `last_seen_at` | 最近一次摄取的 batch.created_at | UPSERT（每次） | 采集时间戳 |
| `source_updated_at` | ITR 返回的问题单更新时间 | INSERT + UPSERT | ITR 响应字段 |
| `quality_flag` | 清洗质量标记 | UPSERT 后 | 清洗流程 |
| `previous_status` | ticket_version 关联的上一版 status | INSERT ticket_version | ticket.status（旧值） |

### 3.5 数据库文件路径与连接参数

| 参数 | 值 | 说明 |
|---|---|---|
| 数据库文件路径 | `data/ptm-tse.db` | 可配置，默认相对路径 |
| Journal 模式 | WAL | 支持并发读 |
| 外键约束 | PRAGMA foreign_keys = ON | 每次连接启用 |
| 超时 | 5s | 锁等待超时 |

`data/.gitignore` 内容：
```
*.db
*.db-journal
*.db-wal
*.db-shm
snapshots/
```

### 3.6 受限数据存储策略

> 来源：HLD REV-03 §可信分析治理约束 — 敏感字段策略（"raw 不进 Git/process"）和 §推荐边界与数据模型（"禁止覆盖历史/写 Git/process"）。
> CP5 评审意见 B4 要求 `.gitignore` 之外补充可执行的存储权限落点。

| 配置项 | 值 | 说明 |
|---|---|---|
| 数据根目录 | `data/` | 所有持久化数据的根目录 |
| 子目录 | `data/snapshots/` | 原始 ITR 响应快照（JSON 文件），由 ST-RA-05.1-INGEST 写入 |
| 数据库文件 | `data/ptm-tse.db` | SQLite 规范化数据，WAL 模式 |
| 目录权限（硬断言） | **`0700`**（仅 owner 可读写执行）| 创建 `data/` 和 `data/snapshots/` 时必须 `os.chmod(path, 0o700)`；权限不满足 → 拒绝摄取 |
| 文件权限（硬断言） | **`0600`**（仅 owner 可读写）| `.db`、`.db-wal`、`.db-shm`、快照 JSON 文件创建后必须验证为 `0600`；不满足 → 删除本次未提交数据，停止写入 |
| 运行账户 | 当前用户（不要求专用服务账户）| 首版 MVP |
| Git 排除 | `*.db`, `*.db-journal`, `*.db-wal`, `*.db-shm`, `snapshots/` | `.gitignore` 规则 + CP6 验证 |
| process/ 排除 | `data/` 目录整体不进 `process/` | HLD 要求 |
| 保留策略 | **手动保留、无自动清除** | 所有 snapshot 和 DB 数据持久保留，无自动过期/删除 |
| raw_json 字段 | 数据库存储完整原始 JSON，分析 Skill 只读取清洗后字段 | HLD §敏感字段策略 |
| 敏感字段阻断 | 未经 `field-mapping.yaml` 分类的字段 → 阻断进入 cleaned 分析字段；仅可保留于受限 raw 存储，不进 LLM/报告正文 | 见 ST-RA-05.2-CLEAN LLD |

**实现检查清单（CP6 前置安全条件）**：

1. 创建 `data/` 目录时执行 `os.chmod(path, 0o700)` → 权限不满足则 **拒绝摄取**
2. 创建 `data/snapshots/` 目录时执行 `os.chmod(path, 0o700)` → 权限不满足则 **拒绝摄取**
3. 创建 `data/ptm-tse.db` 后验证 `os.stat(path).st_mode & 0o777 == 0o600` → 不满足则 **删除文件并停止写入**
4. WAL/SHM 文件（`*-wal`, `*-shm`）创建后必须显式验证 `0600`，不得假设自动继承权限；不满足则关闭连接、回滚事务并停止写入
5. 快照 JSON 先写入同目录 `*.tmp`，设置并验证 `0600` 后再原子替换；任一权限校验失败则删除本次 `.tmp` 和未提交快照、回滚事务并停止
6. `data/.gitignore` 确保 `*.db`、`*.db-journal`、`*.db-wal`、`*.db-shm`、`snapshots/` 被排除
7. `git ls-files data/ | grep -E '(^data/snapshots/|\.(db|db-journal|db-wal|db-shm)$$)'` 输出为空
8. fixture 验证 `raw_json` 字段内容不在分析报告模板输出中出现
9. fixture 验证未分类字段被阻断清洗（见 ST-RA-05.2-CLEAN LLD）
10. 首版保留策略：**手动保留、无自动清除**，在 `IngestionQualityReport` 中记录 `data/` 磁盘用量和 snapshot 数量

## 4. 核心流程

### 4.1 S1 首次摄取写入流程

```
1. conn = get_connection() + PRAGMA foreign_keys=ON
2. begin_transaction(conn)
3. FOR each cleaned ticket record:
   a. 尝试 insert_ticket(ticket_dict)
      - source_ticket_id 已存在 → IntegrityError → 跳过（已在批次中处理）
   b. version = get_next_version(ticket_id)  # 首次为 1
   c. insert_ticket_version(ticket_id, version, None, field_diffs_json, batch_ref)
   d. insert_change_history(ticket_id, batch_ref, 'new', None, 'auto_merged')
4. insert_ingestion_batch(batch_dict)
5. commit(conn)
6. 异常: rollback(conn) + 记录错误 + 不覆盖已有数据
```

### 4.2 S2 增量摄取写入流程

```
1. conn = get_connection() + PRAGMA foreign_keys=ON
2. begin_transaction(conn)
3. FOR each cleaned/compared ticket record:
   a. existing = get_ticket_by_source_id(source_ticket_id)
   b. IF existing IS None:
      - insert_ticket() → version=1 + change_type='new'
   c. ELSE IF field_diff IS EMPTY:
      - change_type='unchanged' → 不写入 ticket_version（无变更）
      - insert_change_history(ticket_id, batch_ref, 'unchanged', None, None)
   d. ELSE:
      - upsert_ticket() → 更新变更字段 + last_seen_at
      - version = get_next_version(ticket_id)
      - insert_ticket_version(ticket_id, version, previous_status, field_diffs_json, batch_ref)
      - insert_change_history(ticket_id, batch_ref, 'modified', affected_fields_json, 'auto_merged')
4. insert_ingestion_batch(batch_dict)
5. commit(conn)
6. 异常: rollback(conn) + 记录错误 + 批次不保存
```

### 4.3 冲突场景流程（主流程中对无 stable ID 或变更冲突的处理）

```
IF source_ticket_id IS NULL OR source_ticket_id 缺失:
  → 不执行 upsert
  → 创建受控 conflict queue 条目，得到 conflict_ref
  → insert_change_history(ticket_id=None, batch_ref, 'conflict', None, 'manual_queue', conflict_ref)
  → 返回冲突项，交由人工处理队列
```

## 5. 状态机

本 Story 中 ticket 自身无业务状态机（ticket 是数据实体，非流程实体）。quality_flag 的取值和转换由 ST-RA-05.2-CLEAN 的清洗流程管理，不在 DAO 层维护。

`IngestionQualityReport.overall_status` 的三态由清洗模块（ST-RA-05.2-CLEAN）写入 `quality_report_ref`，DAO 只被动存储：

```
clean → 所有质量指标达标
flagged → 存在警告但不阻断（部分记录有问题）
blocked → analyzable_ratio 低于阈值，阻止 analysis_run 创建
```

## 6. 错误处理与降级

| 错误场景 | 处理策略 | 降级行为 | 恢复方式 |
|---|---|---|---|
| `source_ticket_id UNIQUE` 冲突 | `IntegrityError` 捕获 → 已有记录跳过，不中止批次 | 批次中其他记录正常写入 | 由调用方在 S2 流程中识别为重复 |
| `batch_id UNIQUE` 冲突 | `IntegrityError` 捕获 → 批次记录跳过错 | 批次已存在，幂等跳过 | 不重复写入；返回已有 batch |
| `ticket_version UNIQUE(ticket_id,version)` 冲突 | `IntegrityError` 捕获 → 回滚当前事务 | 整个批次写入回滚 | 检查 version 号生成逻辑，修正后重试 |
| FK 约束违反（ticket_id 不存在） | `IntegrityError` 捕获 → 回滚当前事务 | 批次不保存 | 检查 ticket 写入是否先于 version/change_history |
| `sqlite3.OperationalError`（锁/磁盘满） | 捕获 → 关闭连接 → 回滚事务 | 批次不保存，已有数据安全 | 检查磁盘空间和锁状态后重试 |
| 数据库文件路径不可访问 | `get_connection()` 抛出异常 → 终止 | 无写入 | 修正路径权限后重试 |
| `field_diffs` JSON 解析失败 | `json.JSONDecodeError` 捕获 → 记录原始值 | 该条记录标记异常 | 修正字段映射后重试 |

**降级原则**：
- 任何写入异常 → 事务回滚 → 不产生部分写入
- 不自动重试（SQLite 是本地文件，不应有瞬态错误）
- 所有异常记录详细错误信息（异常类型、受影响表、batch_ref）

## 7. 测试设计

### 7.1 Fixture 计划

| Fixture | 内容 | 覆盖场景 |
|---|---|---|
| `fixtures/db/empty.db` | 空 SQLite 数据库 | T-ING-09（schema 执行） |
| `fixtures/db/sample_tickets.json` | 5 条有效 ticket 记录 | T-ING-10（CRUD） |
| `fixtures/db/sample_versions.json` | 3 条 ticket 的多版本记录 | T-ING-11（版本递增） |
| `fixtures/db/duplicate_tickets.json` | 相同 source_ticket_id 的记录 | T-ING-12（唯一性） |
| `fixtures/db/corrupt_db.sql` | 故意触发 FK/UNIQUE 冲突的 DML | T-ING-15（回滚） |

### 7.2 测试用例与覆盖矩阵

| ID | 场景 | 类型 | 输入 | 预期结果 | 验证入口 |
|---|---|---|---|---|---|
| T-DB-01 | 执行 DDL 创建所有表 | positive | `schema.sql` | 6 张表 + 13 个索引创建成功 | `sqlite3 :memory: < data/schema.sql` |
| T-DB-02 | INSERT ticket 成功 | positive | 完整 ticket dict | 返回 id > 0 | `insert_ticket()` |
| T-DB-03 | source_ticket_id 重复 → UNIQUE 冲突 | boundary | 相同 source_ticket_id | 抛出 IntegrityError | `insert_ticket()` 重复调用 |
| T-DB-04 | UPSERT 已有 ticket | positive | 已有 ticket + field_diffs | 更新字段 + last_seen_at 刷新 | `upsert_ticket()` |
| T-DB-05 | ticket_version 新增版 | positive | ticket_id=1, version=2 | 写入成功，version 递增 | `insert_ticket_version()` |
| T-DB-06 | ticket_version 版本号重复 | boundary | ticket_id=1, version=2（重复） | 抛出 IntegrityError | `insert_ticket_version()` 重复调用 |
| T-DB-07 | ingestion_batch 写入 | positive | 完整 batch dict | 返回 id | `insert_ingestion_batch()` |
| T-DB-08 | change_history 写入 | positive | ticket_id + change_type | 返回 id | `insert_change_history()` |
| T-DB-09 | 事务提交后数据持久化 | positive | 多表写入 → commit | 所有表数据可见 | 关闭重连 + SELECT |
| T-DB-10 | 事务异常回滚 | negative | DML 中触发 IntegrityError | 回滚后所有表无新增数据 | rollback + COUNT(*) |
| T-DB-11 | FK 约束：version 引用不存在的 ticket | boundary | ticket_version(ticket_id=999) | IntegrityError | 无对应 ticket 写入 |
| T-DB-12 | quality_flag 非法值 | boundary | quality_flag='invalid' | ValueError（输入校验层） | validate_ticket() |
| T-DB-13 | 空 source_ticket_id | boundary | source_ticket_id='' | ValueError | validate_ticket() |
| T-DB-14 | get_tickets_by_time_range 过滤 | positive | start < end | 返回时间窗口内记录 | DAO 查询 |
| T-DB-15 | analysis_run 表 DDL 创建 | positive | `schema.sql` | analysis_run 表存在，约束生效 | `sqlite3 :memory: < data/schema.sql` + `PRAGMA table_info(analysis_run)` |
| T-DB-16 | measure_link 表 DDL 创建 | positive | `schema.sql` | measure_link 表存在，CHECK 约束生效 | `sqlite3 :memory: < data/schema.sql` + `PRAGMA table_info(measure_link)` |
| T-DB-17 | 无稳定 ID 冲突留痕 | negative | `ticket_id=None` + `conflict_ref` | change_history 写入成功，ticket 不新增 | DAO fixture |
| T-DB-18 | 措施刷新提示字段 | positive | `proposed_status` + `refresh_hint` | 仅提示字段更新成功，正式 status 不变 | DAO fixture |
| T-DB-19 | 非 reviewer 修改正式状态 | security | 调用系统提示更新接口传入 status | 拒绝更新 | DAO fixture |

### 7.3 测试执行方式

```python
# 使用临时数据库进行单元测试
conn = sqlite3.connect(':memory:')
conn.executescript(Path('data/schema.sql').read_text())
# 执行各 DAO 函数测试
```

## 8. 安全与权限

| 安全要求 | HLD 来源 | 实现方式 | 验证方式 |
|---|---|---|---|
| 受限数据不进 Git | `IngestionQualityReport` 契约、HLD §敏感字段策略 | `data/.gitignore` 排除 `*.db`、`*.db-journal`、`*.db-wal`、`*.db-shm`、`snapshots/` | `git ls-files data/ | grep -E '(^data/snapshots/|\.(db|db-journal|db-wal|db-shm)$$)'` 输出为空 |
| raw_json 不进 LLM/正文 | HLD §敏感字段策略 | 数据库存储在 `raw_json` 列，分析 Skill 只读取清洗后字段 | fixture 验证 raw_json 字段不在报告输出中 |
| DAO 单写 / 受限调用 | HLD §推荐边界与数据模型 | 只有 F-020 写入 `data/dao.py`；F-021/F-023 仅能调用已定义的公共 DAO 方法，不得直接 SQL 或新增方法 | 代码审查：`data/dao.py` 仅由 F-020 修改；调用方仅使用获授权接口 |
| SQLite WAL 模式 | 行业最佳实践 | `PRAGMA journal_mode=WAL` 在 `get_connection()` 中设置 | fixture 验证 WAL 文件创建 |
| 外键约束强制 | 数据完整性 | `PRAGMA foreign_keys=ON` 每次连接启用 | fixture 验证 FK 违规抛出 IntegrityError |

**不做的事**：
- 不实现 SQL 注入防护（使用参数化查询，由 DAO 调用方确保——Python sqlite3 的 `?` 占位符天然防止注入）
- 不加密数据库文件（首版 MVP，加密待后续安全 CR）
- 不实现连接池（单文件本地数据库，并发需求低）

## 9. 实施步骤

### 切片 1：DDL + .gitignore（TASK-DB-01, TASK-DB-05）
1. 创建 `data/` 目录
2. 写入 `data/.gitignore`
3. 写入 `data/schema.sql`（DDL + 索引）
4. 验证：`sqlite3 :memory: < data/schema.sql` 无语法错误

### 切片 2：连接管理 + Ticket CRUD（TASK-DB-02）
1. 实现 `get_connection()`（WAL + FK）
2. 实现 `validate_ticket()` 输入校验
3. 实现 `insert_ticket()` + `upsert_ticket()` + `get_ticket_by_source_id()`
4. 单元测试：T-DB-01..04, T-DB-12..14

### 切片 3：Version + Batch + Change History（TASK-DB-03..05）
1. 实现 `insert_ticket_version()` + `get_next_version()`
2. 实现 `insert_ingestion_batch()` + `get_batch()`
3. 实现 `insert_change_history()`
4. 单元测试：T-DB-05..08

### 切片 4：事务管理 + 回滚（TASK-DB-06）
1. 实现 `begin_transaction()` / `commit()` / `rollback()`
2. 实现批量写入包装函数（事务包裹）
3. 异常路径测试：T-DB-09..11

### 验证汇总
- 12 个单元测试全部通过
- `data/schema.sql` SQL 语法无错误
- `data/.gitignore` 生效（`git status` 不显示 `*.db`）
- CP6 检查通过后交付验证

## 10. 回滚策略

| 变更类型 | 回滚方式 | 注意事项 |
|---|---|---|
| DDL 错误（`schema.sql`） | 新版 DDL + 迁移脚本；或删除 `data/ptm-tse.db` 重建 | 若已有生产数据需提供 `ALTER` 迁移脚本 |
| DAO 逻辑错误 | 回退 `data/dao.py` 到上一版本 | DAO 接口变更需同步更新调用方 |
| 数据库文件损坏 | 从 SQLite `.backup` 恢复；或从原始快照重新摄取重建 | 定期备份策略待独立 CR |
| `.gitignore` 规则遗漏 | 追加排除规则 + `git rm --cached` 已跟踪的 `.db`/`.db-shm`/`.db-wal` 文件 | 不得 force push 删除数据库文件 |

**回滚触发条件**：
- CP7 验证发现数据完整性问题（约束未生效、事务回滚失败）
- 下游 Feature 报告 schema 不兼容
- 数据库文件意外进入 Git 跟踪

## 11. 平台差异检查

| 检查项 | Codex | Claude Code | Qoder | 说明 |
|---|---|---|---|---|
| Python sqlite3 模块可用 | 取决于运行环境 | 取决于运行环境 | 取决于运行环境 | 本 LLD 产物（SQL + DAO）是纯文件，不依赖特定平台 |
| SQLite 版本 | ≥3.35.0 | ≥3.35.0 | ≥3.35.0 | WAL 模式 + RETURNING 子句需 3.35+ |
| 文件路径分隔符 | `/` (POSIX) | `/` (POSIX) | `/` (POSIX) | `pathlib.Path` 保证跨平台 |
| .gitignore 规则 | 标准 git | 标准 git | 标准 git | 无平台差异 |

**结论**: 无平台差异处理项。`data/schema.sql`、`data/dao.py`、`data/.gitignore` 均为平台无关的纯文本/代码文件。

## 12. 与相邻模块的集成契约

### 12.1 对 FEAT-RA-ANALYSIS（ST-RA-05.3-ANALYZE）的承诺

| 承诺 | 实现 | 约束 |
|---|---|---|
| ticket 数据只读查询 | `get_tickets_by_product()`, `get_tickets_by_time_range()`, `get_tickets_by_batch()` | 不修改 ticket 行 |
| version 历史可追溯 | `get_ticket_versions()` 返回完整历史 | version 只增不减 |
| batch 关联可用 | `get_batch()` 返回 `quality_report_ref` | batch 写入后不可变 |
| quality_flag 与 batch 绑定 | 每次清洗后 quality_flag 记录在 ticket 行 | ANALYSIS 消费时检查 quality_flag |
| analysis_run 写入 | `insert_analysis_run()` → `update_analysis_run_draft()`；`reviewer_publish_analysis_run()` | ANALYSIS 仅维护草案执行状态；发布须 reviewer 调用专用接口 |
| analysis_run 查询 | `get_analysis_run()`, `get_runs_by_batch()` | 按 run_id 或 batch_ref 检索 |

### 12.2 对 FEAT-RA-TRACKING（ST-RA-04/06.3-TRACK）的承诺

| 承诺 | 实现 | 约束 |
|---|---|---|
| measure_link 写入 | `insert_measure_link()`, `update_measure_refresh_hint()`, `reviewer_update_measure_status()` | TRACKING 仅调用 DAO；系统只写提示字段，正式状态和基线仅 reviewer 可变更 |
| measure_link 查询 | `get_measure_link()`, `get_measures_by_status()` | 按 proposal_ref 或 status 检索 |

### 12.3 data/dao.py 单写原则

- **F-020 是 `data/dao.py` 的唯一写入者**：所有 DAO 方法（含 analysis_run 和 measure_link）在 Wave 1 一次性实现。
- F-021 和 F-023 只通过 `import data.dao` 调用公共接口，不修改此文件；允许的业务写入仅限已列明的 analysis_run / measure_link 方法。
- **CP4 文件所有权**：`data/dao.py` 仅列在 ST-RA-INGEST-DB 的 output_files 中；后续 Story 不拥有此文件的写入权。
- 若后续需要新增 DAO 方法，必须通过 CR 修改 ST-RA-INGEST-DB 或在独立 DAO 文件中实现，不得追加到 `data/dao.py`。

- SQLite WAL 模式下，单写者 + 多读者是安全并发模型
- 若多个进程同时写入，SQLite 会自动串行化（`busy_timeout=5000`）
- 生产环境中写操作应由 `itr-ticket-ingestion` Skill 独占执行

## 13. 开放项与假设

### 开放项

| ID | 描述 | 状态 | 影响 | 决议条件 |
|---|---|---|---|---|
| LCQ-ST-RA-INGEST-DB-01 | analysis_run 和 measure_link 表的 DDL 与 DAO 归属 | ✅ RESOLVED（2026-07-16） | F-020 创建全部 6 张表 DDL 并一次性实现全部 DAO CRUD（含 analysis_run 和 measure_link 的读写方法）；F-021/F-023 仅调用公共接口，不修改 `data/dao.py`。物理 DAO 写入统一由 F-020 承载；业务语义归各 Feature（analysis_run 归 F-021，measure_link 归 F-023）。host-orchestrator 选方案 A。 | 已回填 |
| O-DB-01 | SQLite 数据库文件默认路径 `data/ptm-tse.db` 是否合适 | OPEN | DAO 默认参数 | CP5 确认或用户指定 |
| O-DB-02 | 大规模写入（>10k records）的性能表现 | OPEN（延后至性能 CR） | batch 写入策略 | 实测后决定是否批量提交 |
| O-DB-03 | 首版保留策略为"手动保留、无自动清除"；定期备份和自动清除待独立安全/合规 CR | OPEN（延后至安全/合规 CR） | 数据量增长后磁盘占用 | 安全/合规 CR 批准后补充备份/清除机制 |

### 假设

| ID | 假设 | 依据 | 风险 |
|---|---|---|---|
| A-DB-01 | SQLite 数据库文件由 `itr-ticket-ingestion` Skill 独占写入 | HLD §推荐边界与数据模型：SQLite 数据 owner 的单写 | 中（若多进程写入可能触发 busy timeout） |
| A-DB-02 | 首版不需要加密和访问控制 | HLD 非功能需求：静态文件协议，不授权运行时 | 低（本地文件权限足够） |
| A-DB-03 | source_ticket_id 长度不超过 256 字符 | 典型 ITR 工单 ID 格式（如 TKT-2026-00001） | 低 |

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD：4 表 DDL + DAO + 14 章节 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 评审 B1 整改：扩展为 6 表 DDL（新增 analysis_run + measure_link），DAO 方法归属明确。CP5 评审 B4 整改：新增 §3.6 受限数据存储策略（权限/备份/清除/敏感字段阻断）。 |
| 1.2 | 2026-07-16 | host-orchestrator | CP5 Round 2 整改：F-020 为 data/dao.py 唯一写入者，一次性定义全部 6 表 DAO（含 analysis_run + measure_link CRUD）。下游 Feature 只调用接口，不修改此文件。§3.6 权限从"推荐"改为硬断言（0700/0600 + 失败清理）。新增 priority 列。 |
| 1.3 | 2026-07-16 | host-orchestrator | CP5 Round 3 整改：§13 LCQ 旧结论修正（"DAO 方法由各 Feature 追加"→"F-020 一次性实现全部 DAO CRUD，F-021/F-023 仅调用公共接口"）；DDL 注释区分"逻辑领域 owner"与"物理 DDL/DAO 写入 owner"；§1/§3.5/§3.6/§8/§10 全量补充 `*.db-shm` 到 Git 排除、权限硬断言、CP6 检查和回滚策略。 |
| 1.4 | 2026-07-16 | host-orchestrator | CP5 Round 4 整改：统一快照 `0700/0600` 硬断言与 Git 排除；修正无稳定 ID 冲突的可持久化留痕；补齐 `measure_link.proposed_status/refresh_hint` 与 reviewer-only 正式状态接口；澄清下游仅调用公共 DAO、不直接 SQL 的权限边界。 |
| 1.5 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
