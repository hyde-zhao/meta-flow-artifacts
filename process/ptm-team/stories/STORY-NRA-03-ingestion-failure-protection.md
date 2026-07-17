---
story_id: "ST-NRA-03"
title: "摄取失败保护（异常/冲突/保存失败）"
status: "ready-for-verification"
priority: "P0"
wave: 1
source_cr: "CR-030"
source_story: "ST-NRA-03"
feature: "FEAT-RA-INGESTION"
feature_design_refs:
  - "docs/features/feat-ra-ingestion/DESIGN.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: []
  rationale: "负向场景，正向 Story 的 LLD 已覆盖失败路径设计，Story 卡片内补充技术说明"
depends_on:
  - "ST-RA-05.1-INGEST"
  - "ST-RA-05.2-CLEAN"
output_files:
  - "skills/itr-ticket-ingestion/SKILL.md"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-NRA-03: 摄取失败保护

## dev_context

### 背景
在 itr-ticket-ingestion Skill 中追加失败路径处理。实现 FEAT-RA-INGESTION DESIGN.md §4.3 中定义的四种失败场景保护。

### 技术说明

失败保护需在正向摄取的每个关键步骤插入守卫：

1. **HTTP 失败**：响应非 2xx → 记录 http_status + error → 不创建 batch，不写入任何 ticket
2. **Schema 不匹配**：必填字段缺失或结构错误 → batch blocked → 不写入 ticket
3. **SQLite 写入失败**：使用事务包裹，异常时完整回滚
4. **无稳定 ID**：source_ticket_id 缺失 → 进入人工冲突队列 → 不自动合并

所有失败场景必须输出可恢复的错误说明（失败原因 + 受影响范围 + 重试条件），且不覆盖历史快照和已有 ticket 数据。

## validation_context

### 验证入口
- 使用 `fixtures/itr_malformed_response.json` 测试 schema 不匹配
- 模拟 SQLite 写入异常测试事务回滚

### 关键验证场景
1. HTTP 502 → batch 不创建
2. schema 不匹配 → 不写入 ticket
3. SQLite 写入异常 → 事务回滚，已有数据不变
4. 无 source_ticket_id → 人工队列记录

## acceptance_criteria

1. HTTP 失败不覆盖历史快照
2. schema 不匹配不写入 ticket
3. 写入失败时事务完整回滚
4. 无稳定 ID 时进入人工队列，不自动合并
5. 每种失败输出可恢复错误说明

## 技术说明

> `lld_policy.required_level=technical-note`：本 Story 为负向保护场景，其失败路径设计依据和验证入口已由正向 Story LLD（ST-RA-05.1-INGEST §6、ST-RA-05.2-CLEAN §6、ST-RA-06.1-DETECT §6）覆盖。此处仅记录设计决策摘要和与相邻 Story 的关系，不另出独立 LLD。

### 设计决策摘要

本 Story 在 `skills/itr-ticket-ingestion/SKILL.md` 中追加 §失败保护段，覆盖 4 个关键失败场景的守卫逻辑：

| 失败场景 | 守卫位置 | 守卫动作 | 设计依据 |
|---|---|---|---|
| HTTP 非 2xx | ST-RA-05.1 的 `fetch_itr_tickets()` 之后 | 抛出 `HTTPFetchError(recoverable=False)` → 不创建 batch → 不写入 ticket → 返回错误: `{http_status, error, recoverable}` | LLD §6 错误处理与降级 |
| Schema 不匹配（必填字段缺失） | ST-RA-05.2 的 `clean_tickets()` 中必填字段校验 | 标记 `FailedRecord('missing_required')` → quality_report.overall_status='blocked' → 不写入 ticket | LLD §4 清洗主流程、LLD §6 |
| SQLite 写入失败 | ST-RA-06.1 的批量写入事务 | `rollback(conn)` → 批次全部不保存 → 返回错误详情 | LLD §4 写入流程、LLD §6 |
| 无 source_ticket_id | ST-RA-06.1 的 `detect_changes()` 入口 | 进入 `conflict_queue`（reason='missing_id'）→ 不自动合并 → conflict-queue.yaml 输出供人工处理 | LLD §4.1 变更检测、LLD §2.3 |

### 与正向 Story 的关系

- **ST-RA-05.1-INGEST LLD §6**：定义了 HTTP 摄取层面的全部错误类型（`HTTPFetchError`、`AllowlistDeniedError`、`CredentialDeniedError`、`HTTPMethodDeniedError`、`SnapshotSaveError`）及其降级策略。本 Story 消费这些错误类型，增设错误输出格式和可恢复性标记。
- **ST-RA-05.2-CLEAN LLD §6**：定义了清洗层面的 `FailedRecord` 处理（必填字段缺失 → 不进入 cleaned、不写入 ticket）。本 Story 将其与 batch blocked 逻辑衔接。
- **ST-RA-06.1-DETECT LLD §6**：定义了变更检测层面的冲突处理（无 stable ID → manual_queue）和事务回滚。本 Story 将其提升为用户可见的失败保护语义。

### 为何不需要完整 LLD

1. **失败路径是正向流程的异常分支**：4 个失败场景分别附着在正向摄取管线的 3 个步骤中，不需要独立的模块、数据模型或流程设计。
2. **设计决策已在正向 LLD 中完整定义**：每个失败场景的守卫位置、触发条件、错误类型、降级行为和恢复方式已在正向 Story LLD 的 §6（错误处理与降级）和 §4（核心流程）中完整覆盖。
3. **验证入口明确**：每个失败场景在正向 Story 的测试矩阵中已有对应测试用例（T-ING-10/11, T-CLN-04/09/10, T-DET-05/10），本 Story 不新增独立的测试用例。
4. **文件影响范围单一**：仅修改 `skills/itr-ticket-ingestion/SKILL.md`（追加 §失败保护），不涉及新文件、新数据表或新接口。

### 实施要点

- 在 SKILL.md 的 §失败保护段中，以检查清单（checklist）形式列出 4 个守卫点及其失败行为
- 每个守卫点输出结构化错误：`{scenario, error_type, recoverable, affected_scope, retry_conditions}`
- SKILL.md 中应包含失败保护的"停止条件"：任一步失败不继续后续步骤
- 依赖 ST-RA-05.1-INGEST 和 ST-RA-05.2-CLEAN 已完成其正向摄取的 §错误处理段（即错误类型已定义）

### 风险与重访条件

| 风险 | 重访条件 |
|---|---|
| ITR 真实响应中的失败模式超出设计假设 | 首次受控探测后发现新的 HTTP 错误码或 schema 变体 |
| 事务回滚对大规模批次的影响未验证 | 性能 CR 中测试 >1000 条记录的批次回滚 |

### LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-dev | 初始 technical-note：失败保护设计决策摘要、与正向 Story 关系、豁免理由 |
