---
feature_id: "FEAT-17"
change_id: "CR-157"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-05T13:18:00+08:00"
owner: "host-orchestrator"
implementation_allowed: false
---

# Mature Admission Package Tasks

## CP5 Design Tasks

| Task ID | Owner Story | Task | Exit Criteria |
|---|---|---|---|
| CR157-F17-T01 | CR157-S01 | Define package object model, mandatory ref families and status enum. | Full LLD lists fields, statuses and serialization rules. |
| CR157-F17-T02 | CR157-S01 | Define completeness and fail-closed policy. | Missing/stale/invalid ref outcomes are explicit and testable. |
| CR157-F17-T03 | CR157-S01 / S02 | Define evidence-index linkage. | Package entries reference evidence ids; no embedded body fields. |
| CR157-F17-T04 | CR157-S04 | Define forbidden operation counters and blocked behavior. | Counter list covers all no-runtime/no-write boundaries. |

## Deferred Tasks

| Deferred ID | Reason | Unlock Condition |
|---|---|---|
| DF-CR157-001 | Event adapter implementation is outside CR157 first slice. | Future event-specific CR and CP3/CP5 approval. |
| DF-CR157-002 | ML adapter implementation is outside CR157 first slice. | Future ML-specific CR and CP3/CP5 approval. |
