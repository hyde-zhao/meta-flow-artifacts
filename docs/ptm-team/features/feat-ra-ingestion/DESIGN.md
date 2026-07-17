---
status: draft
version: "1.0"
source_cr: "CR-030"
source_hld: "docs/design/HLD.md (v1.2, REV-03)"
feature_id: "FEAT-RA-INGESTION"
agent: "ptm-tse"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-INGESTION: 问题单摄取与数据治理 — Feature 设计

> 对应 HLD REV-03 的 `itr-ticket-ingestion` Skill + SQLite Data Owner 边界。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-se | 初始 Feature 设计。 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 Round 4：SQLite 物理 owner 一次性定义 6 表及公共 DAO；统一原始快照权限/Git 排除；补足冲突留痕与 reviewer-only 状态契约。 |

---

## 1. Feature 概述

### 目标

让测试架构师从用户指定的 ITR 来源安全、可控地获取现网问题单，经过清洗和质量管理后存入版本化 SQLite 数据库，为后续的逆向分析和改进跟踪提供唯一、可信、可追溯的数据基础。

### 成功标准

| 指标 | 度量方式 | 目标值 |
|---|---|---|
| 摄取安全 | deny-by-default：0 个凭据读取、ITR 写入、外部系统访问路径 | 100%（fixture 证明） |
| 数据可追溯 | 每条 ticket 均可回链到原始快照、HTTP 请求参数和拉取时间 | 100% |
| 质量可见 | 每个 ingestion_batch 关联 IngestionQualityReport | 100% |
| 失败保护 | schema 不匹配/保存失败时不覆盖历史数据 | 100%（fixture 证明） |
| 版本化 | 每次 upsert 保留 ticket_version 和 change_history | 100% |

### 非目标

- 不连接 ITR 以外的外部系统（TAC、日志、工单、知识库）
- 不推断或读取认证凭据
- 不向 ITR 发出写操作
- 不将原始数据写入 Git 或 process 目录
- 不生成分析结论或 CA/PA 建议

---

## 2. 模块架构

```
User Request (product, time_range, pagination)
    │
    ▼
┌──────────────────────────────────────┐
│ itr-ticket-ingestion Skill            │
│                                       │
│  ┌─────────┐  ┌─────────┐  ┌───────┐ │
│  │ HTTP GET │─▶│ Clean/  │─▶│Quality│ │
│  │+Snapshot │  │  Map    │  │Report │ │
│  └─────────┘  └─────────┘  └───────┘ │
│       │              │          │     │
│       ▼              ▼          ▼     │
│  ┌──────────────────────────────────┐ │
│  │       SQLite Data Owner          │ │
│  │  ticket | ticket_version | ingestion_batch │ │
│  │  change_history | analysis_run | measure_link    │ │
│  └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

---

## 3. 关键接口

### 3.1 HTTP 摄取接口（ITR GET）

| 属性 | 值 |
|---|---|
| 方法 | GET only |
| URL | 用户指定，须通过 allowlist 校验 |
| 参数 | product, time_range, pagination（只允许白名单参数） |
| 认证 | 无凭据、无认证头推断 |
| 重试 | 不自动重试；失败保留错误记录 |
| 超时 | 30s（可配置） |

**allowlist 规则**：
- URL 必须匹配预配置的 allowlist pattern
- 参数名必须在白名单内
- 任何写入方法（POST/PUT/PATCH/DELETE）直接拒绝

### 3.2 SQLite Schema（DDL）

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
    raw_json TEXT NOT NULL,       -- 原始响应 JSON
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
    field_diffs TEXT NOT NULL,    -- JSON: {field: {old, new}}
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
    change_type TEXT NOT NULL,    -- 'new' | 'modified' | 'unchanged' | 'conflict'
    affected_fields TEXT,          -- JSON array
    resolution TEXT,               -- 'auto_merged' | 'manual_queue' | 'rejected'
    conflict_ref TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    CHECK (ticket_id IS NOT NULL OR conflict_ref IS NOT NULL)
);

-- 分析运行表（逻辑 owner: FEAT-RA-ANALYSIS；物理 DDL/DAO owner: FEAT-RA-INGESTION）
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

-- 措施关联表（逻辑 owner: FEAT-RA-TRACKING；物理 DDL/DAO owner: FEAT-RA-INGESTION）
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

### 3.3 IngestionQualityReport 结构

```yaml
batch_id: string
schema_version: string
input_summary:
  total_fetched: int
  http_status: int
  response_hash: string
quality_summary:
  total_clean: int
  total_flagged: int
  total_failed: int
  null_rate: float          # 空值比例
  anomaly_rate: float       # 异常值比例
  duplicate_rate: float     # 重复率
  conflict_count: int       # 冲突数
  data_classification: enum  # 'confirmed' | 'partial' | 'unknown'
overall_status: enum        # 'clean' | 'flagged' | 'blocked'
analyzable_ratio: float     # 可分析比例
issues:
  - field: string
    type: enum              # 'null' | 'anomaly' | 'duplicate' | 'conflict' | 'unclassified'
    count: int
    description: string
```

---

## 4. 关键流程

### 4.1 S1 首次摄取流程

```
1. 用户请求 → allowlist 校验 → 拒绝或通过
2. 发起 HTTP GET → 校验 HTTP 状态 → 失败则 record failure
3. 保存原始快照（raw response + metadata）到非 Git 受限数据区
4. 字段映射：ITR fields → ticket schema fields
5. 清洗：
   - 空值/异常值检测 → quality_flag
   - 未知字段保留原始引用（不静默丢弃）
   - source_ticket_id 唯一性校验 → 重复记录
6. 质量报告生成：IngestionQualityReport
7. 写入 SQLite：
   - INSERT ticket（source_ticket_id 去重）
   - INSERT ticket_version (version=1)
   - INSERT ingestion_batch
   - INSERT change_history (type='new')
8. 返回摄取摘要 + 质量报告引用
```

### 4.2 S2 增量摄取 + 变更检测流程

```
1. 新批次拉取（同 S1 步骤 1-5）
2. 按 source_ticket_id 匹配已有 ticket：
   - 新 ID → type='new'
   - 已有 ID → 字段级 diff：
     - 无变化 → type='unchanged'
     - 有变化 → type='modified' + field_diffs JSON
     - 无稳定 ID → 人工冲突队列
3. 冲突处理：
   - 可自动合并（字段级非冲突变更）→ resolution='auto_merged'
   - 不可自动合并（语义冲突）→ resolution='manual_queue'
4. 写入 SQLite：
   - UPSERT ticket（更新 last_seen_at、source_updated_at、变更字段）
   - INSERT ticket_version（新 version 号）
   - INSERT change_history（type + resolution）
5. 返回变更摘要：新增 N、修改 M、冲突 K
```

### 4.3 失败路径

| 失败场景 | 处理 | 影响 |
|---|---|---|
| HTTP 非 2xx | 记录 http_status + error，不重试 | 批次失败，已有数据不受影响 |
| Schema 不匹配 | 记录 mismatch 详情，不写入 ticket | 批次 blocked |
| 字段缺失（必填） | 标记缺失字段，quality_flag='incomplete' | 该条记录不可分析，其他记录正常 |
| 数据分类未确认 | quality_report.status='flagged' | 批次可保存但标注风险 |
| SQLite 写入失败 | 事务回滚，记录失败原因 | 批次全部不保存 |

---

## 5. 安全约束

| 规则 | 实现方式 | 验证方式 |
|---|---|---|
| 无凭据读取 | Skill 不声明 credential 工具、不解析认证头 | forbidden-request fixture |
| 仅 allowlist URL | URL pattern 白名单；非匹配拒绝 | URL allowlist fixture |
| 仅 GET 方法 | 方法白名单；POST/PUT/DELETE 拒绝 | HTTP method fixture |
| 原始数据不进 Git | `data/.gitignore` 排除 `*.db`、`*.db-journal`、`*.db-wal`、`*.db-shm`、`snapshots/` | git-tracking fixture |
| 受限存储 | `data/` / `data/snapshots/` 为 `0700`，DB/WAL/SHM/JSON 为 `0600`；失败清理本批次未提交数据并停止 | permission-failure fixture |
| 敏感字段分类 | 未分类字段不进 LLM/正文；分类词典版本化 | sensitive-field fixture |
| 无 ITR 写入 | Skill 无写操作声明 | skill-capability review |

---

## 6. 数据模型细节

### 6.1 去重策略

- 主去重键：`source_ticket_id`
- 同键首次摄取：创建新 ticket（version=1）
- 同键后续摄取：
  - 字段无变化：不更新 ticket，change_history='unchanged'
  - 字段有变化：upsert ticket + 新增 ticket_version + change_history='modified'
- 无 source_ticket_id 或键冲突：进入人工处理队列，不自动合并

### 6.2 版本化策略

- 每次导致 ticket 字段变更的摄取创建一个新 ticket_version
- version 号单调递增
- 旧版本不覆盖，通过 ticket_id + version 唯一索引保留
- ticket 主表始终保留最新状态 + first_seen_at

### 6.3 质量阈值（LLD 版本化）

清洗阈值由 LLD 定义，analysis_run 引用版本。候选阈值：

| 参数 | 建议默认值 | 可配置 |
|---|---|---|
| null_rate_threshold | 0.3（30%） | 是 |
| anomaly_rate_threshold | 0.1（10%） | 是 |
| duplicate_rate_threshold | 0.5（50%） | 是 |
| min_analyzable_ratio | 0.5（50%） | 是 |
| max_conflict_count | 50 | 是 |

---

## 7. 与相邻 Feature 的契约

### 7.1 对 FEAT-RA-ANALYSIS 的承诺

- 提供 ticket/version/batch 的只读查询，以及 ANALYSIS 的 analysis_run 草案写入公共接口
- ticket 记录的 quality_flag 与 IngestionQualityReport 绑定
- 每次 analysis_run 可引用 ingestion_batch.batch_id 和 schema_version
- 不清楚或未分类的字段通过 quality_flag 标注，不会静默删除

### 7.2 对 FEAT-RA-IMPROVEMENT 的承诺

- 不越过 ANALYSIS 直接向 IMPROVEMENT 提供数据
- IMPROVEMENT 通过 ANALYSIS 间接消费 ticket 数据

### 7.3 对 FEAT-RA-TRACKING 的承诺

- TRACKING 通过公共 DAO 查询 `measure_link`；系统仅可写刷新提示字段，reviewer 才能创建 baseline 或改变正式状态

---

## 8. Gotchas

- **不要把 HTTP GET 成功等同于数据可信**：清洗和质量标记是必需的中间层
- **不要自动重试 HTTP 请求**：ITR 是外部系统，重试策略需要用户明确
- **不要静默丢弃未知字段**：未知字段必须保留原始引用，在 quality_flag 中标注
- **不要把 source_ticket_id 缺失当作可恢复错误**：这是阻断条件，须进入人工队列
- **不要让任何受限数据进入 Git**：`.gitignore` 必须排除 SQLite 主/辅助文件和 `snapshots/`，并在 fixture 中验证
- **不要低估 schema drift**：ITR 响应 schema 可能随时变化，每次拉取必须校验
