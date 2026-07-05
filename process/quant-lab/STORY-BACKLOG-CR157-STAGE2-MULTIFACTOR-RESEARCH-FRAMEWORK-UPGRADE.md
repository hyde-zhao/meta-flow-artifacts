---
change_id: "CR-157"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-05T13:18:00+08:00"
owner: "host-orchestrator"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
implementation_allowed: false
---

# CR157 Story Backlog

## Story Overview

| Story | Title | Wave | lld_policy | Depends On |
|---|---|---|---|---|
| CR157-S01 | Mature admission package builder contract | W1 | full-lld | none |
| CR157-S02 | Research evidence index traceability | W2 | full-lld | S01 |
| CR157-S03 | Stage 2 to Stage 3 handoff hardening | W3 | full-lld | S01, S02 |
| CR157-S04 | No-runtime guard coverage | W4 | full-lld | S01, S02, S03 |
| CR157-S05 | Docs, release wording and backlog alignment | W5 | technical-note | S01, S02, S03, S04 |

## Dependency DAG

```text
S01 -> S02 -> S03 -> S04 -> S05
S01 --------> S03
S01 --------> S04
S02 --------> S04
S01 --------> S05
S02 --------> S05
S03 --------> S05
```

## File Ownership Summary

| Story | Primary Future Files | Parallel Dev Policy |
|---|---|---|
| S01 | `engine/mature_multifactor_framework.py`, `tests/research/test_mature_multifactor_framework.py` | Serial merge owner for package contract. |
| S02 | Same shared engine/test file plus future evidence index output | Serial after S01 to avoid schema drift. |
| S03 | Same shared engine/test file | Serial after S02 to consume package/evidence contract. |
| S04 | Same shared engine/test file | Serial after S03 because guard counters affect all statuses. |
| S05 | docs/product, docs/design, docs/components, docs/release targets | Serial after S04 so wording reflects final guard semantics. |

## Not Authorized

CP4 does not authorize LLD approval, implementation, tests implementation, real lake/NAS/provider/QMT/runtime/simulation/paper/live/trading/broker/credential/feed/order/reconciliation/store/catalog/registry/publish/external framework/Git remote write.
