---
feature_id: "FEAT-15"
change_id: "CR-154"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-03T10:40:00+08:00"
owner: "host-orchestrator"
implementation_allowed: false
---

# Cross-Strategy Reliability Gates Task Plan

## CP5 Design Tasks

| Task ID | Story | Task | Output |
|---|---|---|---|
| CR154-T01 | S01 | Design shared gate summary, artifact ref and blocked-claim schema. | Full LLD field contract and first runnable fixture plan. |
| CR154-T02 | S02 | Design Gate 1 statistical artifact model, severity mapping and adapter subtasks. | Full LLD with 12 artifact slots and strategy-family subtask table. |
| CR154-T03 | S03 | Design Gate 2 CV governance contract. | Full LLD with walk-forward/OOS/purged-embargo mapping. |
| CR154-T04 | S04 | Design Gate 3 PIT universe contract and CR153 slot lifecycle. | Full LLD with delegated-to-CR154 compatibility semantics. |
| CR154-T05 | S05 | Design Gate 4 capacity/impact/liquidity contract. | Full LLD with controlled impact enum and no-real-TCA claim boundary. |
| CR154-T06 | S06 | Design Gate 5 regime/attribution/reconciliation slots. | Full LLD with slot/status/ref/n/a validation and no-runtime reconciliation boundary. |
| CR154-T07 | S07 | Design Gate 6 admission tier resolver and release wording. | Full LLD with fail-closed unknown profile behavior. |
| CR154-T08 | S08 | Design compatibility and follow-through wording. | Technical note with exact artifact targets and deferred scope mapping. |

## Future Implementation Task Families

These are planning anchors only. They are not authorized by CP4.

| Family | Candidate Files | Verification Entry |
|---|---|---|
| Shared contract | `engine/cross_strategy_reliability_gates.py` | `tests/research/test_cross_strategy_reliability_gates.py` |
| Admission package integration | `engine/strategy_admission_package.py` | `tests/research/test_strategy_admission_package.py` plus CR154 tests |
| Strategy adapters | `engine/strategy_admission_statistical_gate.py`, `engine/ml_strategy_admission_gate.py`, `engine/event_strategy_admission_gate.py` | Existing CR151/152/153 tests plus adapter fixtures |
| Release/process evidence wording | `process/returns/CR154-*.return.json`, `process/evidence/CR154-*.index.json`, CP7/CP8 CR154 artifacts | CP7/CP8 checks after implementation |

## Merge Order

1. S01 shared contract and fixture skeleton.
2. S02-S06 gate-specific policies may be designed in parallel after S01, but implementation touching the shared contract should merge in story-id order unless CP5 assigns separate file sections.
3. S07 admission policy waits for S02-S06 status semantics.
4. S08 compatibility and wording waits for S02-S07.

## Not Authorized

CP4 does not authorize any implementation task. Any real data, runtime, broker, feed, reconciliation, store/catalog/registry, publish or Git remote operation requires a separate explicit authorization gate.
