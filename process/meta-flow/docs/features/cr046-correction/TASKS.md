---
status: draft
version: "1.0"
feature_id: "FEAT-EI-CORRECTION"
---

# Feature Tasks: Correction Lifecycle and Pilot Adapter

| TASK-ID | 顺序 | 任务 | 输出范围 | 验证 | 状态 |
|---|---:|---|---|---|---|
| TASK-EI-007-01 | 1 | 定义CorrectionEvent schema/allowlist/supersedes checker | workflow/schema/tests | CT-COR-01..04 | pending |
| TASK-EI-007-02 | 2 | 实现append-only correction lifecycle与rollback | workflow/cli/tests | CT-COR-01/04/08 | pending |
| TASK-EI-007-03 | 3 | 实现CR-163 adapter dry-run/pilot manifest授权边界 | adapter/tests | CT-COR-05/06 | pending |
| TASK-EI-007-04 | 4 | 实现legacy codex_agent_name D3 migration/replay | adapter/fixtures | CT-COR-07 | pending |
