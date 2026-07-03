---
status: draft
version: "1.0"
feature_id: "FEAT-PG-003"
feature_name: "Project State Governance"
source_design: "process/docs/features/project-state-governance/DESIGN.md"
source_test_plan: "process/docs/features/project-state-governance/TEST-PLAN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Tasks: Project State Governance

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| PSG-TASK-001 | 1 | 定义 `PROJECT.current.json` schema、16KB 预算、forbidden fields 和错误码。 | HLD ADR-PG-002、DESIGN 数据模型。 | schema/checker 模块、fixtures。 | primary | PSG-UNIT-01..03 | pending |
| PSG-TASK-002 | 2 | 定义 `PROJECT-SCALE.yaml` schema，明确 gate profile bias 与不授权项。 | DESIGN PSG-DQ-002。 | scale schema、negative fixtures。 | primary | PSG-UNIT-04、PSG-SEC-03 | pending |
| PSG-TASK-003 | 3 | 定义 `ROADMAP.yaml` / `MILESTONES.yaml` baseline schema 与 reader contract。 | HLD §18、DESIGN roadmap/milestone 数据模型。 | roadmap/milestone schema、reader typed snapshot。 | primary | PSG-UNIT-05、PSG-CON-01 | pending |
| PSG-TASK-004 | 4 | 实现 project scaffold dry-run/apply 计划和冲突保护。 | DESIGN scaffold flow。 | scaffold service / CLI、tmp workspace tests。 | primary | PSG-INT-01..03、PSG-SEC-01 | pending |
| PSG-TASK-005 | 5 | 接入 FEAT-PG-001 current-state writer 写入 `project_state_ref`。 | FEAT-PG-001 writer contract、DESIGN PSG-IF-03。 | writer integration code、contract tests。 | shared | PSG-CON-02、PSG-SEC-04 | pending |
| PSG-TASK-006 | 6 | 实现 project-state checker，覆盖预算、断 ref、ID 重复和状态枚举。 | schema / fixtures。 | checker result、finding model。 | primary | PSG-INT-04、PSG-CON-01 | pending |
| PSG-TASK-007 | 7 | 更新 Story 卡片和 CP5 设计证据引用，确保 CR-C stories 指向本设计包。 | Feature Matrix、DESIGN Story 拆分建议。 | process/stories 下游卡片更新计划。 | shared | CP4 / CP5 precheck | pending |
| PSG-TASK-008 | 8 | 补充人工验收说明，覆盖 scale bias、refs-only 和不覆盖用户文件。 | TEST-PLAN 手工验收。 | QA checklist / manual evidence。 | shared | PSG-MAN-01..03 | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| PSG-BLK-001 | PSG-TASK-005 | FEAT-PG-001 writer/checker 若尚未实现，无法完成 current ref 写入集成。 | host-orchestrator / meta-se | 在 Story DAG 中要求 FEAT-PG-001 先于 CR-C-S01 的 writer integration。 |
| PSG-BLK-002 | PSG-TASK-007 | CP5 尚未批准，不能进入实现或最终 Story 执行。 | host-orchestrator / user | 保持本任务清单为 draft baseline，等待 CP5。 |
