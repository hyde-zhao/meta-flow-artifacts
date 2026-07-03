---
status: draft
version: "1.0"
feature_id: "FEAT-PG-007"
feature_name: "Roadmap Follow-up Tracking"
source_design: "process/docs/features/roadmap-follow-up-tracking/DESIGN.md"
source_test_plan: "process/docs/features/roadmap-follow-up-tracking/TEST-PLAN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Tasks: Roadmap Follow-up Tracking

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| TASK-FU-001 | 1 | 固化 FU-RF / SP-RF / RA-RF regex 与命名示例。 | ADR-PG-006、本 DESIGN | 后续 LLD 确认的 tracking regex / tests | primary | UNIT-FU-01 | pending |
| TASK-FU-002 | 2 | 设计 candidate 最小字段、状态枚举和模板。 | Domain OBJ-PG-014 | 后续 LLD 确认的 template / fixture | primary | CONTRACT-FU-01 / UNIT-FU-02 | pending |
| TASK-FU-003 | 3 | 更新 tracking status query，使 roadmap follow-up 可按前缀和状态查询。 | TASK-FU-001/002 | 后续 LLD 确认的 query 模块 | primary | INTEG-FU-02 | pending |
| TASK-FU-004 | 4 | 设计 RELEASE-CONTEXT guardrail，禁止 FU-RF/SP-RF/RA-RF 写入。 | RULE-PG-010 | guardrail fixture / checker contract | shared | SEC-FU-01 | pending |
| TASK-FU-005 | 5 | 设计 roadmap refresh result 消费契约。 | FEAT-PG-006 result schema | consumer contract fixture | shared | INTEG-FU-01 / CT-FU-001 | pending |
| TASK-FU-006 | 6 | 在 CP5 前审查 CR-G-S01 是否保持 technical-note 或升级 full-lld。 | 文件影响面、Story draft | lld_policy decision note | none | MAN-FU-03 | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| BLK-FU-001 | TASK-FU-006 | CR-G-S01 的真实文件影响面尚未在 LLD 中确认。 | host-orchestrator / meta-se | 默认 technical-note；若触及多个 tracking 模块、新增 schema 或 release guardrail，则升级 full-lld。 |
| BLK-FU-002 | TASK-FU-004 | RELEASE-CONTEXT guardrail 需要定位 release-readiness 当前入口。 | meta-dev / meta-qa | LLD 阶段读取相关模块并锁定文件所有权。 |
