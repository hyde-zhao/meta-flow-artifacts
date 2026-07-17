---
checkpoint: "CP6"
story_id: "ST-RA-INGEST-DB"
canonical_story_id: "ST-RA-INGEST-DB"
title: "SQLite 数据库 schema 与 DAO 层 — CP6 编码完成检查"
result: "PASS"
checked_at: "2026-07-16"
checked_by: "meta-dev"
---

# CP6-ST-RA-INGEST-DB-sqlite-schema-dao-CODING-DONE

## Entry Criteria
- [x] Story `ST-RA-INGEST-DB` status=`planned`, LLD v1.5 confirmed
- [x] `dev_gate` satisfied: 无依赖 Story，Wave 1 首个执行
- [x] 设计证据 `ST-RA-INGEST-DB-LLD.md` v1.5 已确认
- [x] 文件所有权不冲突（F-020 独占写入 `data/dao.py`、`data/schema.sql`、`data/.gitignore`）

## 实现证据

### 实现对象清单

| 文件 | 操作 | 行数 | 说明 |
|------|------|------|------|
| `data/schema.sql` | 新建 | 175 | 6 张表 DDL + 13 个索引 + 所有约束 |
| `data/dao.py` | 新建 | 929 | 34 个公共函数：连接管理、CRUD、事务、校验、存储初始化 |
| `data/.gitignore` | 新建 | 5 | 排除 `*.db`、`*.db-journal`、`*.db-wal`、`*.db-shm`、`snapshots/` |
| `.gitignore` | 修改 | +3 | 添加 `!data/.gitignore`、`!data/schema.sql`、`!data/dao.py` 排除否定 |

### 设计契约映射

| LLD 章节 | 实现对应 |
|----------|---------|
| §2.1 DAO 公共接口（31 个方法签名） | `data/dao.py` 全部实现，参数和返回类型匹配 |
| §2.3 输入校验（6 个字段 + 3 个校验函数） | `validate_ticket()`, `validate_change_type()`, `validate_resolution()` |
| §3.1 DDL（6 张表） | `data/schema.sql` 逐字符对齐 |
| §3.2 索引（13 个） | 全部创建，含 `IF NOT EXISTS` |
| §3.3 约束清单（15 项） | UNIQUE, NOT NULL, DEFAULT, CHECK, FK 全部实现 |
| §3.6 受限存储策略（10 项检查清单） | `init_storage()` + `_ensure_dir_with_perms()` + `_ensure_file_perms()` + `.gitignore` |
| §4 核心流程（S1/S2/冲突） | DAO 函数组合支持三种流程 |
| §6 错误处理（8 种场景） | 参数化查询防注入 + 异常逐层传播 + 事务回滚 |
| §8 安全与权限 | WAL + FK + 参数化查询 + 权限硬断言 |
| §10 回滚策略 | 事务 begin/commit/rollback 显式接口 |
| §12 集成契约 | F-021/F-023 的公共接口方法均已实现 |

### 验证结果

| 验证项 | 方法 | 结果 |
|--------|------|------|
| DDL 语法 | Python sqlite3 :memory: 执行 `schema.sql` | PASS: 6 表 + 13 索引创建成功 |
| Ticket CRUD | insert/upsert/get_by_source_id/get_by_product/get_by_time_range/get_by_batch | PASS |
| UNIQUE 约束 | source_ticket_id 重复插入 | PASS: IntegrityError |
| Ticket Version | insert/get_versions/get_next_version + UNIQUE 冲突 | PASS |
| Ingestion Batch | insert/get/get_by_time | PASS |
| Change History | insert (正常/冲突留痕) + get_by_ticket/get_by_batch | PASS |
| 无稳定 ID 冲突 | ticket_id=None + conflict_ref 留痕 | PASS: ValueError 正确拒绝无 conflict_ref 场景 |
| Analysis Run | insert/get/get_by_batch/update_draft/reviewer_publish | PASS: draft 方法拒绝 published |
| Measure Link | insert/update_refresh_hint/reviewer_update_status/get/get_by_status | PASS: 系统提示拒绝修改 status |
| 事务回滚 | BEGIN + INSERT + ROLLBACK | PASS: 回滚后数据不可见 |
| 事务提交 | BEGIN + INSERT + COMMIT | PASS: 提交后数据可见 |
| FK 约束 | ticket_version 引用不存在的 ticket_id | PASS: IntegrityError + 自动回滚 |
| 输入校验 | 空 source_ticket_id、非法 quality_flag、过长 id | PASS: ValueError |
| init_storage | 临时目录 0700 权限 | PASS |
| .gitignore | git check-ignore 验证源文件追踪、DB 文件排除 | PASS |
| 参数化查询 | 全函数 review | PASS: 所有 SQL 使用 `?` 占位符 |

### 平台差异处理

N/A — `data/schema.sql`、`data/dao.py`、`data/.gitignore` 均为平台无关的纯文本/代码文件。使用 `pathlib.Path` 保证跨平台路径兼容。

### 未覆盖项

| 项 | 原因 |
|----|------|
| 大规模写入性能测试 (>10k records) | LLD O-DB-02 延后至性能 CR |
| 数据库加密 | LLD 明确首版不做 |
| 生产备份恢复 | LLD O-DB-03 延后至安全/合规 CR |
| WAL/SHM 文件权限实时验证 | 依赖 SQLite 自动创建，需在实际文件系统测试（非 :memory:） |

### 设计缺口反馈

无。所有 LLD 要求的功能均已在 CP5 确认的范围内实现。

## Exit Criteria
- [x] 所有输出文件非空且路径正确
- [x] 设计契约映射完整
- [x] 所有 25 项功能测试通过
- [x] .gitignore 规则生效（源文件可追踪，运行时文件被排除）
- [x] 参数化查询防 SQL 注入
- [x] 异常回滚不产生部分写入
- [x] 代码注释使用中文，函数名/变量名使用英文
- [x] 使用 Python 标准库 sqlite3，无 ORM
- [x] 使用 pathlib.Path 跨平台兼容

## 后续交接

- **给 meta-qa 的验证入口**: `uv run python -c "import sqlite3; c = sqlite3.connect(':memory:'); c.executescript(open('data/schema.sql').read()); print('DDL OK')"`
- **风险提示**: 无。全部功能在临时内存数据库中验证通过。
