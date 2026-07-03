---
status: draft
version: "1.0"
feature_id: "FEAT-PG-005"
feature_name: "Impact Surface Normalization"
source_design: "process/docs/features/impact-surface-normalization/DESIGN.md"
source_test_plan: "process/docs/features/impact-surface-normalization/TEST-PLAN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Tasks: Impact Surface Normalization

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| ISN-TASK-001 | 1 | 定义 normalized impact schema：`impact_surfaces`、`affected_paths`、`feature_refs`、`capability_refs`。 | DESIGN 数据模型、HLD 成功标准。 | schema、fixtures。 | primary | ISN-UNIT-01 | pending |
| ISN-TASK-002 | 2 | 冻结 impact surface enum 和扩展规则，初始覆盖至少 14 类治理面。 | DESIGN enum、HLD 成功标准。 | enum definition、manual review checklist。 | primary | ISN-UNIT-02、ISN-MAN-02 | pending |
| ISN-TASK-003 | 3 | 接入 FEAT-PG-004 resolver，校验 `feature_refs` / `capability_refs`。 | registry resolver contract。 | refs validation adapter。 | shared | ISN-CON-01、ISN-SEC-02 | pending |
| ISN-TASK-004 | 4 | 实现 new CR audit/enforce checker，覆盖 unknown enum、invalid path 和 authorization misuse。 | schema、mode policy。 | CR checker updates、negative fixtures。 | primary | ISN-INT-01、ISN-INT-02、ISN-SEC-01、ISN-SEC-03 | pending |
| ISN-TASK-005 | 5 | 实现 legacy migration scanner 和 migration report，保证不改写源 CR。 | CR index、legacy field fixtures。 | migration report writer、hash-stability tests。 | primary | ISN-INT-03、ISN-SEC-04 | pending |
| ISN-TASK-006 | 6 | 为 change-impact-analysis 和 checkpoint-manager 建立消费契约。 | normalized schema、checker result。 | consumer contract tests / docs updates。 | shared | ISN-CON-02、ISN-CON-03 | pending |
| ISN-TASK-007 | 7 | 准备 unclassified legacy values 的人工回修流程。 | migration report。 | manual review checklist / follow-up input。 | shared | ISN-MAN-01 | pending |
| ISN-TASK-008 | 8 | 更新 Story 卡片和 CP5 设计证据引用，确保 CR-E-S01 指向本设计包。 | Feature Matrix、DESIGN Story 拆分建议。 | process/stories 下游卡片更新计划。 | shared | CP4 / CP5 precheck | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| ISN-BLK-001 | ISN-TASK-003、ISN-TASK-004 | FEAT-PG-004 resolver 未冻结前，refs 校验只能设计 contract。 | host-orchestrator / meta-se | 在 Story DAG 中要求 CR-D-S01 先于 CR-E-S01 refs enforcement。 |
| ISN-BLK-002 | ISN-TASK-002 | 新增 impact surface enum 需要人工确认，不能由实现阶段自由扩展。 | user / architecture owner | 将 enum 扩展作为 CR 或 CP5 决策项。 |
| ISN-BLK-003 | ISN-TASK-008 | CP5 尚未批准，不能进入实现或最终 Story 执行。 | host-orchestrator / user | 保持本任务清单为 draft baseline，等待 CP5。 |
