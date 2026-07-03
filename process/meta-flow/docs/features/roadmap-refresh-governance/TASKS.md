---
status: draft
version: "1.0"
feature_id: "FEAT-PG-006"
feature_name: "Roadmap Refresh Governance"
source_design: "process/docs/features/roadmap-refresh-governance/DESIGN.md"
source_test_plan: "process/docs/features/roadmap-refresh-governance/TEST-PLAN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Tasks: Roadmap Refresh Governance

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| TASK-RF-001 | 1 | 设计并实现 ROADMAP-REFRESH result schema 与 fixture。 | HLD、ADR-PG-005、本 DESIGN | 后续 LLD 确认的 schema / fixture 文件 | primary | UT-RF-001..003 / CT-RF-004 | pending |
| TASK-RF-002 | 2 | 设计并实现独立 roadmap-refresh checker，包含 wrong-result-type failure。 | TASK-RF-001 | 后续 LLD 确认的 checker 模块 / CLI | primary | UNIT-RF-02 / SEC-RF-03 | pending |
| TASK-RF-003 | 3 | 设计并实现 process-only cascade target guard。 | ADR-PG-003、Dependency Map FD-PG-003 | 后续 LLD 确认的 cascade writer / guard tests | primary | INTEG-RF-01 / SEC-RF-02 | pending |
| TASK-RF-004 | 4 | 设计并实现 Gate Ledger `roadmap_refresh` event append 契约。 | Gate Ledger contract、TASK-RF-001 | 后续 LLD 确认的 ledger event appender / fixture | shared | INTEG-RF-02 / CT-RF-003 | pending |
| TASK-RF-005 | 5 | 设计并实现 stale_items / follow_up_candidates 输出契约。 | FEAT-PG-007 / FEAT-PG-008 contracts | result producer / contract fixture | shared | CT-RF-001 / CT-RF-002 | pending |
| TASK-RF-006 | 6 | 补齐 security failure fixtures，证明不写 release repo、不读凭据、不做 runtime/publish。 | TEST-PLAN SEC-RF-* | security fixtures / review evidence | shared | SEC-RF-01..05 | pending |
| TASK-RF-007 | 7 | 更新 Story LLD 的文件所有权、CLI 名称和验证命令。 | CP5 clarification、TASK-RF-001..006 | CR-F-S01/02 Story LLD | none | CP5 自动预检 | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| BLK-RF-001 | TASK-RF-002 / TASK-RF-007 | checker CLI 最终命名需在 LLD 中锁定。 | host-orchestrator / user | 默认 `meta-flow check roadmap-refresh`，若 CLI 体系已有 project 子命令则在 LLD 调整。 |
| BLK-RF-002 | TASK-RF-003 | 发布库写授权不在当前范围。 | user | 保持 process-only cascade；发布库修复转 FU-RF 或正式 CR。 |
