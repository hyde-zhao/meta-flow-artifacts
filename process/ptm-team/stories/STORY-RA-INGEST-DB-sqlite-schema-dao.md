---
story_id: "ST-RA-INGEST-DB"
title: "SQLite 数据库 schema 与 DAO 层"
status: "planned"
priority: "P0"
wave: 1
source_cr: "CR-030"
source_story: "ST-RA-05"
feature: "FEAT-RA-INGESTION"
feature_design_refs:
  - "docs/features/feat-ra-ingestion/DESIGN.md"
  - "docs/features/feat-ra-ingestion/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "migration"]
  rationale: "SQLite schema 是跨 Feature 共享的唯一数据 owner，DDL 和 DAO 接口需完整 LLD"
depends_on: []
output_files:
  - "data/schema.sql"
  - "data/dao.py"
  - "data/.gitignore"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-INGEST-DB: SQLite 数据库 schema 与 DAO 层

## dev_context

### 背景
本 Story 创建 CR-030 的 SQLite 数据库基础设施。这是四个 Feature 共享的唯一数据 owner，所有后续 Story 依赖此 schema。

### 输入文件
- `docs/features/feat-ra-ingestion/DESIGN.md` — Feature 设计（含完整 DDL）
- `docs/design/HLD.md` (REV-03) — 架构约束
- `docs/design/DOMAIN-MAP.md` — 领域对象定义

### 输出文件
- `data/schema.sql` — 完整 DDL（ticket, ticket_version, ingestion_batch, change_history, analysis_run, measure_link）
- `data/dao.py` — DAO 层（CRUD + 事务管理）
- `data/.gitignore` — 排除 SQLite 主/辅助文件和 `snapshots/` 原始数据目录

### 接口约定
按 FEAT-RA-INGESTION DESIGN.md §3.2 的 DDL 实现。DAO 至少提供：
- `insert_ticket()` / `upsert_ticket()` — 按 source_ticket_id 去重
- `insert_ticket_version()` — 版本递增
- `insert_ingestion_batch()` / `get_batch()`
- `insert_change_history()` / `get_changes_by_ticket()`（无稳定 ID 冲突以 `ticket_id=NULL + conflict_ref` 留痕）
- `insert_analysis_run()` / 草案状态更新 / reviewer 发布接口
- `insert_measure_link()` / 系统刷新提示 / reviewer 正式状态更新接口
- 事务管理（commit/rollback）

### 设计约束
- F-020 是 `data/dao.py` 与 `data/.gitignore` 的唯一物理写入者；下游只调用公共 DAO，不直接 SQL
- SQLite 主/辅助文件和 `snapshots/` 不进 Git
- `data/`、`data/snapshots/` 必须 `0700`；数据库、WAL、SHM 和快照 JSON 必须 `0600`，失败即清理未提交数据并停止
- 所有写操作使用事务
- source_ticket_id 有 UNIQUE 约束
- ticket_version 有 UNIQUE(ticket_id, version)

### AI 可执行任务清单
1. 创建 `data/` 目录和 `.gitignore`
2. 编写 `schema.sql`（6 张表 + 索引）
3. 编写 `dao.py`（CRUD + 事务）
4. 编写基本单元测试验证 DDL 执行正确

## validation_context

### 验证入口
- 执行 `schema.sql` 验证 DDL 无语法错误
- 运行 DAO 单元测试（CRUD + 事务回滚）

### 验证方式
- static: schema.sql 语法检查
- fixture: 使用临时 SQLite 数据库测试 DAO

### 关键验证场景
1. DDL 执行成功，所有表和索引创建
2. source_ticket_id UNIQUE 约束生效
3. ticket_version UNIQUE(ticket_id, version) 约束生效
4. 事务回滚后数据不变

## acceptance_criteria

1. `data/schema.sql` 包含 6 张表的完整 DDL（ticket, ticket_version, ingestion_batch, change_history, analysis_run, measure_link）
2. `data/dao.py` 提供 insert/upsert/select 基本 CRUD 操作
3. `data/.gitignore` 排除 `*.db`、`*.db-journal`、`*.db-wal`、`*.db-shm` 和 `snapshots/`
4. source_ticket_id 重复插入时 UNIQUE 约束报错
5. 事务异常时数据完整回滚
6. 无稳定 ID 的冲突可由 `ticket_id=NULL + conflict_ref` 留痕，不创建 ticket
7. 系统只写 `proposed_status` / `refresh_hint`；正式 measure 状态和 analysis 发布仅 reviewer 可变更
