---
status: draft
version: "1.0"
feature_id: "FEAT-PG-004"
feature_name: "Capability / Feature Registry"
source_design: "process/docs/features/capability-feature-registry/DESIGN.md"
source_test_plan: "process/docs/features/capability-feature-registry/TEST-PLAN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Tasks: Capability / Feature Registry

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| CFR-TASK-001 | 1 | 定义 feature registry YAML schema、ID policy、状态枚举、owner/source refs。 | DESIGN 数据模型、ADR-PG-004。 | registry schema、positive/negative fixtures。 | primary | CFR-UNIT-01 | pending |
| CFR-TASK-002 | 2 | 定义 capability registry YAML schema、feature refs、aliases、deprecated_by。 | DESIGN 数据模型。 | capability schema、fixtures。 | primary | CFR-UNIT-02、CFR-UNIT-04 | pending |
| CFR-TASK-003 | 3 | 实现 registry loader 和 checker，输出统一 finding。 | schema、fixtures。 | loader/checker module。 | primary | CFR-INT-01、CFR-SEC-02 | pending |
| CFR-TASK-004 | 4 | 实现 resolver API，覆盖 resolved、unresolved、deprecated、conflict。 | loader/checker result。 | resolver module、finding model。 | primary | CFR-UNIT-03、CFR-UNIT-05 | pending |
| CFR-TASK-005 | 5 | 建立 impact normalization consumer contract。 | FEAT-PG-005 DESIGN、resolver API。 | contract tests / adapter plan。 | shared | CFR-CON-01、CFR-INT-02 | pending |
| CFR-TASK-006 | 6 | 建立 roadmap refresh、stale-check、migration readiness consumer contract。 | HLD §9、DESIGN 下游消费契约。 | consumer contract tests。 | shared | CFR-CON-02、CFR-CON-03、CFR-INT-03 | pending |
| CFR-TASK-007 | 7 | 设计 candidate report 流程，确保未知样本不写 canonical registry。 | CFR-HG-001、security rules。 | candidate report schema / tests。 | primary | CFR-SEC-01、CFR-SEC-04 | pending |
| CFR-TASK-008 | 8 | 准备人工验收材料，覆盖 ID 命名、owner/source refs、aliases/deprecated。 | TEST-PLAN manual checks。 | manual review checklist。 | shared | CFR-MAN-01..03 | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| CFR-BLK-001 | CFR-TASK-005 | FEAT-PG-005 的 normalized field schema 未冻结前，consumer contract 只能设计到接口层。 | meta-se / lld-designer | 在 CR-D-S01 LLD 中标注对 CR-E-S01 的 schema dependency。 |
| CFR-BLK-002 | CFR-TASK-006 | FEAT-PG-006 / FEAT-PG-008 / FEAT-PG-009 尚未进入 LLD。 | host-orchestrator | 先冻结 resolver API，后续下游按 contract 消费。 |
| CFR-BLK-003 | CFR-TASK-008 | 初始 registry 内容需要人工确认语义归属。 | user / host-orchestrator | CP5 前将命名和 unresolved candidate 纳入人工确认清单。 |
