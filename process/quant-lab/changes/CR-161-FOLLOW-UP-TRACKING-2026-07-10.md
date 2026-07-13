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
| `FU-CR161-001` | Research-engine trial lineage instrumentation | closed (`CR-163`) | CR-163 已关闭；formal CR: `process/changes/CR-163.md`；为未来原生 instrumented runs 提供 lineage foundation。 |
| `FU-CR161-002` | Multiple testing / PBO / DSR computable evidence implementation | closed (`CR-164`) | CR-164 completed CP2-CP8 and closed `READY_WITH_RISK` on 2026-07-12; formal CR: `process/changes/CR-164.md`. |
| `FU-CR161-003` | Walk-forward / OOS evidence producer foundation | closed (`CR-166`) | CR-166 于 2026-07-13 经 CP8 批准以 `READY_WITH_RISK` 关闭；只交付 fixture/static producer foundation，不连接真实数据湖，Stage 3 未启动。 |
| `FU-CR161-004` | Economic cost / impact approximation implementation | candidate | Need net-return, impact, slippage or TCA evidence. |
| `FU-CR161-005` | Capacity / liquidity sizing and alpha decay implementation | candidate | Need capacity curve, ADV participation, liquidity sizing or alpha decay evidence. |
| `FU-CR161-006` | Independent CP7 verifier-lane resilience | candidate | A later high-risk CR needs to rely on CR161 CP7, or the CP8 verifier-independence waiver expires. |
| `FU-CR161-007` | Existing-gate integration and CR155 regression implementation | candidate | C1-C4 producers 的 typed evidence 均稳定后，补齐跨 producer 端到端集成；必须保持 CR155 blocked。 |

These candidates are not active CRs. Starting any item requires a separate CR and explicit authorization.

## Structured activation record

```yaml
follow_up_items:
  - id: FU-CR161-001
    title: Research-engine trial lineage instrumentation
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-163.md
    blocked_by: ""
    next_action: "Closed under CR163; real historical backfill and real ML/event runners remain separately authorized work."
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
  - id: FU-CR161-003
    title: Walk-forward / OOS evidence producer foundation
    kind: implementation-gate
    status: closed
    lifecycle_status: closed
    readiness_status: READY_WITH_RISK
    gate_status: cp8_closed
    gate_profile: architecture-major
    formal_cr_path: process/changes/CR-166.md
    related_cr: CR-166
    blocked_by: ""
    next_action: "Closed under CR166 CP8 approval; real data, runtime and Stage 3 execution remain separately authorized work."
  - id: FU-CR161-004
    title: Economic cost / impact evidence producer
    kind: implementation-gate
    status: candidate
    lifecycle_status: candidate
    readiness_status: not_ready
    gate_status: not_started
    gate_profile: architecture-major
    formal_cr_path: ""
    blocked_by: "C3 input contract and independent authorization"
    next_action: "Keep separate from CR166 computation scope; reuse the future versioned evidence component envelope."
  - id: FU-CR161-005
    title: Capacity / liquidity / ADV / alpha-decay evidence producer
    kind: implementation-gate
    status: candidate
    lifecycle_status: candidate
    readiness_status: not_ready
    gate_status: not_started
    gate_profile: architecture-major
    formal_cr_path: ""
    blocked_by: "C4 input contract and independent authorization"
    next_action: "At CP3 of its future CR, decide whether to share a C3/C4 input-contract wave without merging calculations."
  - id: FU-CR161-006
    title: Independent CP7 verifier-lane resilience
    kind: ledger-maintenance
    status: candidate
    lifecycle_status: candidate
    readiness_status: not_ready
    gate_status: not_started
    gate_profile: process-lite
    formal_cr_path: ""
    blocked_by: "A later high-risk verification route requires it"
    next_action: "Activate only through a separate CR and explicit authorization."
  - id: FU-CR161-007
    title: Existing-gate integration and CR155 regression implementation
    kind: implementation-gate
    status: candidate
    lifecycle_status: candidate
    readiness_status: not_ready
    gate_status: not_started
    gate_profile: architecture-major
    formal_cr_path: ""
    blocked_by: "C1-C4 typed evidence producers are not all available"
    next_action: "Activate after producer contracts stabilize; preserve CR155 blocked and reuse CR151/CR154."
```
