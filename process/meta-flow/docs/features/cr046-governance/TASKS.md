---
status: draft
version: "1.0"
feature_id: "FEAT-EI-GOVERNANCE"
---

# Feature Tasks: Workspace and Lifecycle Governance

| TASK-ID | 顺序 | 任务 | 输出范围 | 验证 | 状态 |
|---|---:|---|---|---|---|
| TASK-EI-004-01 | 1 | 实现共享 RouteTruth 与 real metadata校验 | workspace/routing + tests | CT-GOV-01/02/08 | pending |
| TASK-EI-004-02 | 2 | 收敛 phase/gate/finalization/health/read expansion | state/checks/context + tests | CT-GOV-03..05 | pending |
| TASK-EI-004-03 | 3 | 实现 semantic manifest/restore/apply guard | ledger_compaction + tests | CT-GOV-06/07 | pending |
| TASK-EI-004-04 | 4 | 集成 CLI/doctor/CP共享 resolver | cli/checks + regression | cross-truth一致 | pending |
