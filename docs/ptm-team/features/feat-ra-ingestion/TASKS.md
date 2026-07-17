---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-INGESTION"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-INGESTION — 任务清单

## 交付物清单

| 文件 | 类型 | Story |
|---|---|---|
| `skills/itr-ticket-ingestion/SKILL.md` | Skill 定义 | ST-RA-05.1, 05.2, NRA-03 |
| `skills/itr-ticket-ingestion/templates/quality-report.yaml` | 模板 | ST-RA-05.2 |
| `skills/itr-ticket-ingestion/templates/batch-manifest.yaml` | 模板 | ST-RA-05.1 |
| `data/schema.sql` | SQLite DDL | ST-RA-INGEST-DB |
| `data/dao.py` | DAO 层 | ST-RA-INGEST-DB |
| `data/.gitignore` | Git 排除规则 | ST-RA-INGEST-DB |
| `fixtures/itr_valid_response.json` | 测试 Fixture | ST-RA-05.1 |
| `fixtures/itr_malformed_response.json` | 测试 Fixture | ST-NRA-03 |
| `fixtures/allowlist_config.yaml` | 测试 Fixture | ST-RA-05.1 |

## Story 级任务分解

### ST-RA-05.1-INGEST: ITR 问题单受控摄取与原始快照保存

**TASK-ING-01**: 创建 `skills/itr-ticket-ingestion/` 目录和 SKILL.md 骨架
- 输出: `skills/itr-ticket-ingestion/SKILL.md`（frontmatter + trigger + description）

**TASK-ING-02**: 实现 HTTP GET allowlist 校验
- 包含: URL pattern 白名单、参数白名单、方法限制
- 拒绝: 非白名单 URL → 记录 + 返回

**TASK-ING-03**: 实现 HTTP GET 请求与响应处理
- 包含: 超时控制、状态码校验、响应体保存
- 特点: 无凭据、无认证头推断

**TASK-ING-04**: 实现原始快照保存
- 保存: raw response JSON + request params + timestamp + response hash
- 路径: 非 Git 受限数据区（`.gitignore` 排除）
- 输出: snapshot_ref

**TASK-ING-05**: 调用受限存储初始化与校验
- 前置: ST-RA-INGEST-DB 已创建 `data/.gitignore`
- 校验: 快照目录 `0700`、快照 JSON `0600`、权限失败时清理未提交快照并停止

**TASK-ING-06**: 创建 batch-manifest 模板
- 输出: `skills/itr-ticket-ingestion/templates/batch-manifest.yaml`

### ST-RA-05.2-CLEAN: 字段映射、清洗与质量报告

**TASK-CLEAN-01**: 实现 ITR 字段 → ticket schema 字段映射
- 映射: 标准 ITR 字段 ↔ ticket 表字段
- 处理: 未知字段保留原始引用

**TASK-CLEAN-02**: 实现空值/异常值/重复检测
- 空值: 必填字段缺失 → quality_flag='incomplete'
- 异常: 格式/范围异常 → quality_flag='anomaly'
- 重复: source_ticket_id 重复 → 记录

**TASK-CLEAN-03**: 实现 IngestionQualityReport 生成
- 输出: 符合 schema 的 quality report YAML/JSON
- 包含: 输入摘要、质量摘要、问题清单、overall_status

**TASK-CLEAN-04**: 实现质量阈值判断
- 阈值: null_rate、anomaly_rate、duplicate_rate、analyzable_ratio
- 超标: overall_status='blocked'

**TASK-CLEAN-05**: 实现敏感字段分类检查
- 未分类字段: 不进 LLM/正文，标注 unclassified
- 策略: 分类词典版本化

**TASK-CLEAN-06**: 创建 quality-report 模板
- 输出: `skills/itr-ticket-ingestion/templates/quality-report.yaml`

### ST-RA-INGEST-DB: SQLite 数据库 schema 与 DAO 层

**TASK-DB-01**: 创建 SQLite DDL
- 输出: `data/schema.sql`
- 表: ticket, ticket_version, ingestion_batch, change_history, analysis_run, measure_link
- 包含: PRIMARY KEY, UNIQUE, FOREIGN KEY, 索引

**TASK-DB-02**: 实现 ticket DAO（CRUD）
- INSERT ticket（source_ticket_id 去重）
- UPSERT ticket（版本化更新）
- SELECT by source_ticket_id, product, time_range

**TASK-DB-03**: 实现 ticket_version DAO
- INSERT 新版本（version 号递增）
- SELECT 历史版本 by ticket_id

**TASK-DB-04**: 实现 ingestion_batch DAO
- INSERT batch（batch_id 唯一）
- SELECT by batch_id, time_range

**TASK-DB-05**: 实现 change_history DAO
- INSERT 变更记录（type + resolution；无稳定 ID 冲突使用 `ticket_id=NULL + conflict_ref`）
- SELECT by ticket_id, batch_ref

**TASK-DB-06**: 实现事务管理
- 摄取批次写入使用事务
- 失败时完整回滚

**TASK-DB-07**: 实现跨 Feature 公共 DAO 契约
- analysis_run：ANALYSIS 可创建/更新草案，只有 reviewer 可发布
- measure_link：系统只更新 `proposed_status` / `refresh_hint`，只有 reviewer 可创建 baseline 或改变正式 `status`
- 下游 Feature 只调用公共 DAO，不直接 SQL、不修改 `data/dao.py`

### ST-NRA-03: 摄取失败保护

**TASK-NRA03-01**: 实现 HTTP 失败处理
- 非 2xx → 记录 http_status + error → 不创建 batch

**TASK-NRA03-02**: 实现 Schema 不匹配处理
- 必填字段缺失或结构不匹配 → batch blocked → 不写入 ticket

**TASK-NRA03-03**: 实现 SQLite 写入失败回滚
- 事务异常 → 完整回滚 → 错误记录

**TASK-NRA03-04**: 实现无稳定 ID 处理
- source_ticket_id 缺失 → 进入人工冲突队列 → 不自动合并

**TASK-NRA03-05**: 创建失败 Fixture
- 输出: `fixtures/itr_malformed_response.json`

## 依赖关系

```
ST-RA-INGEST-DB (schema/DAO)
        ▲
        │ depends on
        │
ST-RA-05.1-INGEST ────────────── ST-RA-05.2-CLEAN
        │                               │
        └───────┬───────────────────────┘
                │ extends (failure paths)
                ▼
          ST-NRA-03
```

- ST-RA-05.1 和 ST-RA-05.2 可部分并行（05.2 需要 05.1 的数据结构定义）
- ST-RA-INGEST-DB 阻塞 05.1 和 05.2 的写入部分（需先有 schema）
- ST-NRA-03 依赖 05.1 和 05.2 的失败路径设计
