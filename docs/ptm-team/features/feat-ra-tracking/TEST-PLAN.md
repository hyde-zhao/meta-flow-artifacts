---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-TRACKING"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-TRACKING — 测试计划

## 1. 测试场景

| ID | 场景 | Story | 类型 | 预期 |
|---|---|---|---|---|
| T-TRK-01 | 行动项状态流转 | ST-RA-04 | positive | not-started→in-progress→done |
| T-TRK-02 | 过期行动项标记 | ST-RA-04 | boundary | 超过 due_date → overdue |
| T-TRK-03 | 所有条件满足 → 关闭 | ST-RA-04 | positive | 四条件全满足才可关闭 |
| T-TRK-04 | 任一条件不足 → 保持开放 | ST-RA-04 | boundary | 输出 residual_risks + follow_up |
| T-TRK-05 | 有效性检查结果 | ST-RA-04 | positive | passed/failed/inconclusive 正确 |
| T-TRK-06 | MeasureBaseline 创建 | ST-RA-06.3 | positive | baseline 含版本/范围/审批引用 |
| T-TRK-07 | 无基线 → needs-baseline | ST-RA-06.3 | boundary | 不判措施失效 |
| T-TRK-08 | 措施刷新只产出提示 | ST-RA-06.3 | security | 不自动改变批准状态 |
| T-TRK-09 | 不自动关闭 RA | ST-RA-04 | security | 无自动关闭路径 |
| T-TRK-10 | 不自动改变批准状态 | ST-RA-06.3 | security | 措施刷新不改变 approval_status |
| T-TRK-11 | 刷新提示持久化契约 | ST-RA-06.3 | integration | `proposed_status` / `refresh_hint` 写入 measure_link，正式 status 不变 |

## 2. Fixture 设计

| Fixture | 用途 |
|---|---|
| `fixtures/action_items.json` | T-TRK-01, 02 |
| `fixtures/closure_conditions.json` | T-TRK-03, 04 |
| `fixtures/effectiveness_check.json` | T-TRK-05 |
| `fixtures/measure_baseline.json` | T-TRK-06, 07, 08 |
