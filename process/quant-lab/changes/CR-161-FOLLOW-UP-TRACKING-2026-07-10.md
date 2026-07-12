---
tracking_id: "CR-161-FOLLOW-UP-TRACKING-2026-07-10"
cr_id: "CR-161"
status: "candidate-tracking"
created_at: "2026-07-10T06:20:54+08:00"
owner: "host-orchestrator"
---

# CR161 Follow-up Tracking

| Candidate ID | Title | Status | Activation Condition |
|---|---|---|---|
| `FU-CR161-001` | Research-engine trial lineage instrumentation | active (`CR-163`) | Activated 2026-07-11; formal CR: `process/changes/CR-163.md`; target is `ExperimentFamilyManifest=present` for future instrumented runs. |
| `FU-CR161-002` | Multiple testing / PBO / DSR computable evidence implementation | closed (`CR-164`) | CR-164 completed CP2-CP8 and closed `READY_WITH_RISK` on 2026-07-12; formal CR: `process/changes/CR-164.md`. |
| `FU-CR161-003` | Walk-forward fold-level metrics and OOS evidence computation | candidate | Need fold-level OOS metrics and split evidence. |
| `FU-CR161-004` | Economic cost / impact approximation implementation | candidate | Need net-return, impact, slippage or TCA evidence. |
| `FU-CR161-005` | Capacity / liquidity sizing and alpha decay implementation | candidate | Need capacity curve, ADV participation, liquidity sizing or alpha decay evidence. |
| `FU-CR161-006` | Independent CP7 verifier-lane resilience | candidate | A later high-risk CR needs to rely on CR161 CP7, or the CP8 verifier-independence waiver expires. |

These candidates are not active CRs. Starting any item requires a separate CR and explicit authorization.

## Structured activation record

```yaml
follow_up_items:
  - id: FU-CR161-002
    title: Multiple testing / PBO / DSR computable evidence implementation
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-164.md
    blocked_by: ""
    next_action: "Closed under CR164 CP8 approval; follow-up boundaries are tracked in process/changes/CR-164-FOLLOW-UP-TRACKING-2026-07-12.md."
```
