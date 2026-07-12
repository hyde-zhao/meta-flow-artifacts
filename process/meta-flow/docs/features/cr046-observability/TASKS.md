---
status: draft
version: "1.0"
feature_id: "FEAT-EI-OBSERVABILITY"
---

# Feature Tasks: Replay, Audit and Cost Observability

| TASK-ID | 顺序 | Story | 任务 | 输出范围 | 验证 | 状态 |
|---|---:|---|---|---|---|---|
| TASK-EI-005-01 | 1 | ST-EI-005 | 定义UsageRecord三态与source adapter | checks/context/tests | CT-OBS-01/02 | pending |
| TASK-EI-005-02 | 2 | ST-EI-005 | 实现CR/phase/attempt/checker聚合 | audit input/tests | CT-OBS-02 | pending |
| TASK-EI-006-01 | 3 | ST-EI-006 | 实现checker registry与双口径replay | replay/cli/tests | CT-OBS-03/04 | pending |
| TASK-EI-006-02 | 4 | ST-EI-006 | 实现generated machine audit | audit/cli/tests | CT-OBS-05 | pending |
| TASK-EI-006-03 | 5 | ST-EI-006 | 实现PC-01..19与A/B判定 | conformance/tests | CT-OBS-06/07 | pending |
| TASK-EI-006-04 | 6 | ST-EI-006 | 实现legacy codex_agent_name D3 strict replay | replay/migration fixtures | CT-OBS-08 | pending |
