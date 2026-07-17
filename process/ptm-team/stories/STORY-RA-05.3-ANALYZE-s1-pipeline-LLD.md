---
story_id: "STORY-RA-05.3-ANALYZE"
canonical_story_id: "ST-RA-05.3-ANALYZE"
title: "S1 逐单与批量分析管线"
status: "lld-draft"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
feature: "FEAT-RA-ANALYSIS"
source_cr: "CR-030"
wave: 2
tier: "Tier-A"
created_by: "meta-dev"
created_at: "2026-07-16"
depends_on: ["ST-RA-02", "ST-RA-05.2-CLEAN"]
shared_fragments:
  - shared_file: "skills/reverse-analysis/SKILL.md"
    serial_group: "reverse-analysis"
    write_section: "S1 pipeline, analysis-run management"
  - shared_file: "skills/reverse-analysis/templates/analysis-run-manifest.yaml"
    serial_group: "reverse-analysis-templates"
    write_section: "AnalysisRunManifest schema"
open_items: []
---

# ST-RA-05.3-ANALYZE LLD: S1 逐单与批量分析管线

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | S1 分析管线、AnalysisRunManifest 与 `full-lld` 约束 |
| Story `ST-RA-05.3-ANALYZE` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 S1 数据链路后半段：清洗后数据 → 六维分析 → 报告/AnalysisRunManifest → SQLite 写入。

---

## 1. 文件影响范围

| 文件 | 操作 | 变更性质 | 所有者 |
|------|------|---------|--------|
| `skills/reverse-analysis/SKILL.md` | 修改 | 在 ST-RA-02 的六维引擎段之后，追加 S1 管线段（§10–§12：数据读取、逐单/批量调度、Manifest 写入） | FEAT-RA-ANALYSIS（shared，写 S1 管线段） |
| `skills/reverse-analysis/templates/analysis-run-manifest.yaml` | 创建 | AnalysisRunManifest 模板 schema | FEAT-RA-ANALYSIS |

**不修改**：`data/schema.sql`（只读消费）、`ra-report.yaml` / `metric-definition.yaml`（ST-RA-02 已建，只读引用）。

---

## 2. 接口设计

### 2.1 S1 管线入口契约

| 接口项 | S1 逐单分析 | S1 批量分析 |
|--------|-----------|-----------|
| **触发** | ptm-tse 指定单个 ticket_id | ptm-tse 指定 batch_id 或时间窗口 |
| **输入** | `source_ticket_id` 或 `ticket.id` | `batch_id` 列表或 SQL WHERE 条件 |
| **前置** | 资格检查通过（ST-RA-01）+ 清洗后数据可用（ST-RA-05.2-CLEAN） | 同左 + 批量内所有 ticket 质量通过 |
| **输出** | 单份 ra-report.yaml 实例 + 单条 analysis_run 记录 | 聚合 ra-report.yaml 实例（batch_summary 类型）+ 一条 analysis_run 记录 |
| **禁止** | 自动确认根因、修改 ticket 表、CA/PA 批准 |

### 2.2 SQLite 读取与受限写入接口

本 Story 只调用 ST-RA-INGEST-DB 已提供的公共 DAO；不得在 Skill 中执行 SQL、追加 DAO 方法或修改 `data/dao.py`：

| 查询类型 | 目标表 | 关键条件 | 用途 |
|---------|--------|---------|------|
| 单 ticket 读取 | `ticket` | `get_ticket_by_source_id()` | 逐单分析 |
| 批量 ticket 读取 | `ticket` + `ticket_version` | `get_tickets_by_batch(batch_ref)`，只返回 `quality_flag='clean'` | 批量分析 |
| 批次元数据读取 | `ingestion_batch` | `get_batch(batch_ref)` | 获取 schema/mapping 版本 |
| 历史窗口数据 | `ticket` | `get_tickets_by_time_range()` | 环比同比（ST-RA-06.2 消费） |
| analysis_run 写入 | `analysis_run` | `insert_analysis_run()` / `update_analysis_run_draft()` | 记录草案分析运行 |

**安全约束**：
- ticket/ticket_version/ingestion_batch 仅读；analysis_run 仅可调用已授权的草案写入接口
- 不读取 `ticket.raw_response` 列
- analysis_run INSERT 是唯一的写入操作，且仅写入分析元数据（不入库分析结论正文至 ticket 表）

### 2.3 AnalysisRunManifest 模板

```yaml
# skills/reverse-analysis/templates/analysis-run-manifest.yaml
analysis_run_id: string       # UUID 或序列号
batch_ref: string             # ingestion_batch.batch_id
comparison_batch_ref: string  # S2 only; S1 时为 null
schema_version: string        # 对应 SQLite schema 版本
mapping_rule_version: string  # 字段映射规则版本
analysis_rule_version: string # 分析规则版本（来自 metric-definition.yaml 的 metric_version）
window:
  start: date                # 分析窗口起始（含）
  end: date                  # 分析窗口结束（含）
recompute_mode: enum          # full | incremental（S1 固定为 full）
report_refs:
  - report_id: string
    type: enum                # single_ticket | batch_summary | difference
    ticket_refs: []           # 关联的 ticket 引用（batch_summary 时列出所有）
status: enum                  # created | in_progress | completed | failed
created_at: datetime
created_by: string            # ptm-tse 调用标识
---

### 2.4 analysis_run 写入契约

```python
insert_analysis_run(conn, {
  "run_id": manifest.analysis_run_id,
  "batch_ref": manifest.batch_ref,
  "comparison_batch_ref": manifest.comparison_batch_ref,
  "schema_version": manifest.schema_version,
  "mapping_version": manifest.mapping_rule_version,
  "rule_version": manifest.analysis_rule_version,
  "time_window_start": manifest.window.start,
  "time_window_end": manifest.window.end,
  "recompute_mode": manifest.recompute_mode,
  "report_refs": manifest.report_refs,
  "metric_versions": manifest.analysis_rule_version,
})
```

| 规则 | 说明 |
|------|------|
| `created_at` 是写入时刻 | 不可复现修改 |
| `status` 初始为 `created` | 六维分析完成后更新为 `completed` 或 `failed` |
| `comparison_batch_ref` S1 为 NULL | S2 时才填对比批次 |
| 写入失败 | 整个 analysis_run 状态标记为 `failed`，不发布报告 |

---

## 3. 数据模型

### 3.1 依赖的 SQLite 表（只读 + 仅 analysis_run 写入）

| 表 | 操作 | 字段 |
|----|------|------|
| `ticket` | 公共 DAO SELECT | `source_ticket_id`, `product`, `module`, `severity`, `status`, `root_cause`, `test_missed_analysis`, `test_missed_phase`, `improvement_measures`, `openeddate`, `resolveddate`, `quality_flag` |
| `ticket_version` | 公共 DAO SELECT | `ticket_id`, `version`, `field_diffs`, `batch_ref` |
| `ingestion_batch` | 公共 DAO SELECT | `batch_id`, `quality_report_ref`, `schema_version` |
| `analysis_run` | 公共 DAO INSERT/UPDATE + SELECT | `run_id`, `batch_ref`, `comparison_batch_ref`, `schema_version`, `mapping_version`, `rule_version`, `time_window_start/end`, `recompute_mode`, `status` |

### 3.2 单次调用内的临时对象

| 对象 | 生命周期 | 内容 |
|------|---------|------|
| `AnalysisContext` | 单次管线执行 | batch_ref, tickets[], analysis_run_id, report_refs[] |
| `TicketAnalysisResult` | 逐单分析中 | ticket_ref, ra_report（single_ticket 类型）, status |
| `BatchAnalysisResult` | 批量分析中 | ticket_results[], aggregated_metrics, pattern_observations |

---

## 4. 核心流程

### S1 逐单分析

```
输入: source_ticket_id + batch_ref
输出: ra-report.yaml（single_ticket）+ analysis_run 记录

1. SQLite 公共 DAO 查询
   1a. `get_ticket_by_source_id(source_ticket_id)`；返回记录必须满足 `quality_flag='clean'`
   1b. 若无结果 → blocked（输出数据缺失）
   1c. `get_batch(batch_ref)` 获取 schema/mapping 版本

2. 资格检查（复用 ST-RA-01）
   2a. eligibility_check(ticket)
   2b. 若 rejected/deferred/blocked → 不创建 analysis_run，输出拒绝原因

3. 证据分类（复用 ST-RA-01）
   3a. classify_evidence(ticket)
   3b. 生成 evidence_lines[] + valid_count

4. 六维分析（复用 ST-RA-02）
   4a. 依次执行：根因 → 产品质量 → 流出 → 漏测 → 改进
   4b. 逐单不支持环比同比（需要比较窗口，属于 S2 或批量分析）
   4c. 生成 ra-report sections

5. 创建 analysis_run
   5a. 生成 analysis_run_id
   5b. `insert_analysis_run()`（recompute_mode = 'full', status = 'created'）
   5c. ra-report.report_id = analysis_run.report_refs[0].report_id

6. 输出
   6a. ra-report.yaml 实例（report_type = single_ticket）
   6b. analysis_run 记录
```

### S1 批量分析

```
输入: batch_id 或受限公共 DAO 的时间窗口参数
输出: ra-report.yaml（batch_summary）+ analysis_run 记录

1. SQLite 批量查询
   1a. `get_tickets_by_batch(batch_id)`（通过版本历史关联，且仅返回 `quality_flag='clean'`）
   1b. 若为空 → blocked

2. 逐单资格检查（批量中每个 ticket）
   2a. 拒绝的 ticket 跳过，在报告中记录 skipped_tickets[]

3. 逐单六维分析（每个 eligible ticket）
   3a. 生成 per-ticket ra-report sections

4. 聚合分析
   4a. 产品质量：模块/严重度聚合 → Pareto + 趋势
   4b. 根因：高频模式识别
   4c. 流出：逃逸模式聚合
   4d. 漏测：PPDCS 归类统计
   4e. 改进：CA/PA 去重与优先级排序（但标记为 draft 待人工确认）
   4f. 环比同比：按 openeddate/resolveddate 窗口聚合

5. 创建 analysis_run（批量）
   5a. report_refs 包含 per-ticket reports + batch_summary report
   5b. recompute_mode = 'full'

6. 输出
   6a. 聚合 ra-report.yaml（batch_summary 类型，含 per-ticket 子引用）
   6b. per-ticket ra-reports（可选单独释放）
   6c. analysis_run 记录
```

---

## 5. 状态机

### AnalysisRun 生命周期

```
created ──► in_progress ──► completed
                │
                └──► failed（任一步骤失败）
```

| 状态 | 含义 | 进入条件 | 退出条件 |
|------|------|---------|---------|
| `created` | analysis_run 记录已创建，未开始分析 | INSERT 完成 | 开始执行分析 → `in_progress` |
| `in_progress` | 分析管线执行中 | 任何分析步骤开始 | 所有步骤完成 → `completed`；任一步骤失败 → `failed` |
| `completed` | 分析完成，ra-report 已生成 | 所有维度分析成功 | 不可回退 |
| `failed` | 分析过程中错误 | 资格拒绝/数据不可用/六维异常 | 不可自动重试；需重新发起新的 analysis_run |

**注意**：此状态机只管理分析运行本身。根因结论的确认状态由 ST-RA-02 的四层状态机管理。

---

## 6. 错误处理与降级

| 场景 | 处理方式 | analysis_run 状态 | 输出 |
|------|---------|-----------------|------|
| ticket 不存在 | 记录错误 + 跳过 | 单 ticket 无，批量中 skipped | 缺失 ticket 清单 |
| quality_flag = 'BLOCKED' | 阻断分析 | blocked（不创建 run） | IngestionQualityReport 引用 |
| batch 无 schema_version | 降级为 "unknown" | in_progress → completed（降级标记） | schema_version = "unknown" |
| SQLite 连接失败 | 终止 | failed | 错误详情 |
| 某维度分析失败 | 跳过该维度，继续其余 | completed（部分完成） | 失败维度标记 skipped + 原因 |
| analysis_run INSERT 失败 | 终止 | —（未创建） | SQLite 写入错误 |

---

## 7. 测试设计

### 7.1 正向测试

| ID | 场景 | 预期 |
|----|------|------|
| T-ANL-12 | S1 逐单分析输出 | ra-report（single_ticket）含 facts/hypotheses/gaps |
| T-ANL-13 | S1 批量分析趋势 | ra-report（batch_summary）含聚合趋势 + 模式识别 |
| T-ANL-11 | AnalysisRunManifest 完整性 | batch_ref, schema_version, window, recompute_mode, report_refs 齐全 |
| T-ANL-S03-01 | 批量中逐单分析正确 | per-ticket sections 各 ticket 独立 |
| T-ANL-S03-02 | analysis_run 写入成功 | SQLite 中可查询 |

### 7.2 负向/边界测试

| ID | 场景 | 预期 |
|----|------|------|
| T-ANL-S03-03 | ticket 不存在 | 返回缺失 ticket 清单 |
| T-ANL-S03-04 | 批量中部分 ticket 资格拒绝 | skipped_tickets[] 记录拒绝原因 |
| T-ANL-S03-05 | 某维度分析失败 | 该维度标记 skipped，其余正常 |
| T-ANL-S03-06 | quality_flag = 'BLOCKED' | blocked，不创建 analysis_run |

### 7.3 Fixture 设计

| Fixture | 覆盖 |
|---------|------|
| `fixtures/s1_single_ticket.json` | T-ANL-12 |
| `fixtures/s1_batch_data.json` | T-ANL-13, T-ANL-11 |
| `fixtures/s1_partial_reject_batch.json` | T-ANL-S03-04 |
| `fixtures/s1_failing_dimension.json` | T-ANL-S03-05 |

---

## 8. 安全与权限

| 检查项 | 要求 |
|--------|------|
| SQLite 读取 | ticket/ticket_version/ingestion_batch 仅经公共 DAO 读取 |
| analysis_run 写入 | 唯一允许的业务写入，且仅调用公共草案接口；发布须 reviewer 专用接口 |
| 不写 ticket 表 | 分析结论不写回 ticket 表 |
| 不创建下游任务 | CA/PA 候选标记 draft，不分发到 improvement-tracker 外部 |
| 报告文件路径 | ra-report 写入 `process/` 或 `data/` 受限目录 |

---

## 9. 实施步骤

### TASK-ID 映射

| TASK-ID | 内容 | 输出位置 |
|---------|------|---------|
| TASK-ANL-15 | 实现 SQLite 只读查询适配 | SKILL.md §10（数据读取段） |
| TASK-ANL-16 | 实现 AnalysisRunManifest 生成与管理 | SKILL.md §11（Manifest 管理段）+ `templates/analysis-run-manifest.yaml` |
| TASK-ANL-17 | 实现逐单分析管线 | SKILL.md §12（逐单管线段） |
| TASK-ANL-18 | 实现批量分析管线 | SKILL.md §12（批量管线段） |
| TASK-ANL-19 | 实现 analysis_run 写入 SQLite | SKILL.md §11（写入子段） |
| TASK-ANL-20 | 创建 analysis-run-manifest 模板 | `templates/analysis-run-manifest.yaml` |

### 执行顺序

```
TASK-ANL-20（模板）─────┐
TASK-ANL-15（查询适配）──┼──► TASK-ANL-16（Manifest 管理）
                         │          │
                         │          ├──► TASK-ANL-17（逐单管线）──► 集成测试
                         │          │
                         │          └──► TASK-ANL-18（批量管线）──►
                         │
                         └──► TASK-ANL-19（analysis_run 写入）──►
```

---

## 10. 回滚策略

| 回滚场景 | 操作 |
|---------|------|
| S1 管线回退 | 回退 SKILL.md 到 ST-RA-02 后状态（保留资格检查 + 六维引擎） |
| analysis_run schema 变更 | 修改 templates/analysis-run-manifest.yaml；analysis_run 表中 schema_version 递增 |
| 需要删除历史 analysis_run | SQLite DELETE（仅可由 F-020 执行；本 Story 只 SELECT/INSERT） |
| 模板文件删除 | 不影响已有 analysis_run 记录引用（历史 manifest 数据已固化在 SQLite） |

---

## 11. 平台差异检查

| 平台 | 差异项 | 影响 | 处理 |
|------|--------|------|------|
| 全部 | SQLite 读写 | 无差异 | 所有平台支持 SQLite |
| 全部 | YAML 模板 | 无差异 | YAML 跨平台一致 |
| 全部 | analysis_run 写入 | 无差异 | SQL INSERT 通用 |

**结论**：无平台差异。

---

## 12. 与相邻模块的集成契约

### 12.1 上游依赖

| 上游 | 输出 | 消费方式 |
|------|------|---------|
| ST-RA-02（六维引擎） | 六个维度的分析方法 | 同 Skill 内直接调用 |
| ST-RA-05.2-CLEAN（清洗后数据） | ticket 表（quality_flag = PASS） | SQLite SELECT |
| ST-RA-INGEST-DB（SQLite schema） | `ingestion_batch` 表结构 | SQLite SELECT（schema_version, mapping_rule_version） |

### 12.2 下游消费

| 下游 | 消费内容 | 传递方式 |
|------|---------|---------|
| ST-RA-03（改进输入治理） | ra-report 中的 improvement_candidates + analysis_run 引用 | 文件引用传递 |
| ST-RA-06.2-REFRESH（S2 增量） | analysis_run 记录 + 历史 ra-reports | SQLite 历史查询 + 文件引用 |
| ST-RA-04（闭环跟踪） | ra-report 中的 analysis_run_id 引用 | 跨 Feature 引用 |

### 12.3 AnalysisRunManifest 格式契约

- `analysis_run_id` 在所有 S1/S2 中全局唯一
- `schema_version`, `mapping_rule_version`, `analysis_rule_version` 必须与创建时一致
- `recompute_mode` 在 S1 中固定为 `full`
- `report_refs[]` 中每条 entry 对应一份 ra-report 文件

---

## 13. 开放项与假设

| ID | 类型 | 描述 | 状态 | 重访条件 |
|----|------|------|------|---------|
| OPEN-RA053-01 | 假设 | analysis_run 的 analysis_run_id 生成策略（UUID vs 序列号） | OPEN | 实现前确定；建议使用 `{batch_id}-{timestamp}` 格式 |
| OPEN-RA053-02 | 假设 | 批量分析中 per-ticket ra-reports 作为独立文件还是内嵌在 batch_summary 中 | OPEN | 建议：内嵌在 batch_summary 中 + 可选独立文件，以 batch_summary 的 report_refs 引用 |
| OPEN-RA053-03 | 假设 | 批量分析的 ticket 数量上限 | OPEN | 建议：无硬上限，但 > 100 时提示性能风险 |

---

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD，覆盖 S1 逐单/批量管线、SQLite 读取契约、AnalysisRunManifest 模板和 analysis_run 生命周期 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 Round 4：改为只调用 F-020 提供的 DAO；统一 manifest→DDL 字段映射、批次查询和 `quality_flag='clean'` 语义，发布改为 reviewer 专用路径。 |
| 1.2 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
