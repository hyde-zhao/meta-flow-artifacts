---
status: draft
version: "1.0"
feature_id: "FEAT-PG-009"
feature_name: "Quant-lab Migration Readiness"
source_design: "process/docs/features/quant-lab-migration-readiness/DESIGN.md"
source_test_plan: "process/docs/features/quant-lab-migration-readiness/TEST-PLAN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Tasks: Quant-lab Migration Readiness

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| TASK-QL-001 | 1 | 设计 migration dry-run report schema 和 readonly observation 字段。 | HLD、Domain OBJ-PG-016、本 DESIGN | 后续 LLD 确认的 schema / fixture | primary | UNIT-QL-02 / CT-QL-* | pending |
| TASK-QL-002 | 2 | 设计 release repo read-only adapter 和 forbidden write guard。 | ADR-PG-003、RULE-PG-012 | security guard / forbidden target fixtures | primary | SEC-QL-01 / SEC-QL-03 / SEC-QL-04 | pending |
| TASK-QL-003 | 3 | 设计 process-side cleanup planned changes、apply authorization 和 rollback refs。 | FEAT-PG-001/003 contracts | process-side writer contract / rollback fixture | primary | INTEG-QL-02 / INTEG-QL-06 | pending |
| TASK-QL-004 | 4 | 设计 capability registry migration adapter，unresolved 时 blocked finding。 | FEAT-PG-004 contract | adapter contract / unresolved fixture | shared | INTEG-QL-03 / CT-QL-002 | pending |
| TASK-QL-005 | 5 | 设计 impact migration adapter 和 unknown surface report。 | FEAT-PG-005 contract | migration report fixture | shared | INTEG-QL-04 / CT-QL-003 | pending |
| TASK-QL-006 | 6 | 设计 stale report 与 FU-RF/正式 CR 候选输出。 | FEAT-PG-006/007/008 contracts | stale report / candidate fixture | shared | INTEG-QL-05 / CT-QL-004 / CT-QL-005 | pending |
| TASK-QL-007 | 7 | 设计人工验收清单，证明未改 quant-lab 发布库代码/tests/正式 docs。 | TEST-PLAN MAN-QL-* | manual acceptance checklist | none | MAN-QL-01..05 | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| BLK-QL-001 | TASK-QL-003 | process-side apply 需要后续 human gate 明确授权；当前仅设计，不授权执行。 | user / host-orchestrator | 默认 dry-run；apply 仅在 CP5/后续实现授权后按 process-side 范围执行。 |
| BLK-QL-002 | TASK-QL-004/005/006 | FEAT-PG-004/005/006/007/008 需先稳定，迁移不能反向污染机制。 | host-orchestrator / meta-se | W5 执行，依赖 W1-W4 验收。 |
| BLK-QL-003 | TASK-QL-002/007 | quant-lab 发布库写入不在当前授权范围。 | user | 保持 read-only；正式修复另开 CR + human gate。 |
