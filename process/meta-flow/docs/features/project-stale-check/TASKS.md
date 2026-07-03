---
status: draft
version: "1.0"
feature_id: "FEAT-PG-008"
feature_name: "Project Stale Check"
source_design: "process/docs/features/project-stale-check/DESIGN.md"
source_test_plan: "process/docs/features/project-stale-check/TEST-PLAN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Tasks: Project Stale Check

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| TASK-ST-001 | 1 | 确认 stale-check CLI 命名、参数和输出摘要契约。 | HLD Q-PG-003、本 DESIGN | CR-G-S02 LLD 中锁定的 CLI contract | none | MAN-ST-01 | pending |
| TASK-ST-002 | 2 | 设计 stale-check result 与 finding schema。 | Domain OBJ-PG-015 | 后续 LLD 确认的 schema / fixture | primary | UNIT-ST-01 / CT-ST-003 | pending |
| TASK-ST-003 | 3 | 设计首批跨对象 stale rules，并排除 cr-tracking 结构规则。 | RULE-PG-011、roadmap/project refs | rule engine contract / fixtures | primary | UNIT-ST-02..04 | pending |
| TASK-ST-004 | 4 | 设计 no-auto-fix guard，禁止自动修改 HLD/TEST/release docs。 | FD-PG-007 | security guard / fixtures | primary | SEC-ST-01 / INTEG-ST-02 | pending |
| TASK-ST-005 | 5 | 设计 actionable finding 到 FU-RF/SP-RF/RA-RF 的 adapter。 | FEAT-PG-007 contract | adapter contract / fixture | shared | INTEG-ST-03 / CT-ST-001 | pending |
| TASK-ST-006 | 6 | 设计 stale-check result 的人工验收摘要和误报处理。 | TEST-PLAN MAN-ST-* | summary format / waive route | shared | MAN-ST-02 / MAN-ST-04 | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| BLK-ST-001 | TASK-ST-001 | stale-check 命令最终命名未确认。 | user / host-orchestrator | 默认 `meta-flow project stale-check`；若项目 CLI 统一走 check 子命令，则切到 `meta-flow check project-stale`。 |
| BLK-ST-002 | TASK-ST-005 | FU-RF tracking 的最终字段需与 FEAT-PG-007 对齐。 | meta-se / meta-dev | 以 FEAT-PG-007 DESIGN 为上游，LLD 阶段锁定 contract。 |
