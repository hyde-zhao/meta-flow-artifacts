---
change_id: "CR-157"
status: "ready-with-risk-draft"
profile: "compact"
created_at: "2026-07-05T14:20:00+08:00"
---

# Feedback: CR157

## Observation Signals

| Signal | Threshold | Route |
|---|---|---|
| Stage 2 package refs cannot map to future real evidence refs | Any missing required ref without `n/a-with-reason` | New CR or design clarification |
| Evidence index needs event-specific fields | More than one event adapter scenario requires fields outside current refs-only item | `DF-CR157-001` |
| Evidence index needs ML-specific fields | More than one ML adapter scenario requires fields outside current refs-only item | `DF-CR157-002` |
| Users interpret CP7 as runtime readiness | Any wording or workflow step implies runtime/publish/trading readiness | Docs/process hygiene follow-up |
| `CR-INDEX.yaml` reappears | Any file matching `process/changes/CR-INDEX.yaml` or `.yml` | Immediate hygiene fix |

## Follow-Up Candidates

| ID | Type | Status |
|---|---|---|
| `DF-CR157-001` | Event adapter implementation | Deferred |
| `DF-CR157-002` | ML adapter implementation | Deferred |
| `FU-CR157-TEST-STRATEGY-SCOPING` | Process hygiene | Candidate |

## Feedback Intake Rule

Feedback here does not start a CR by itself. Host Orchestrator must route accepted feedback through follow-up tracking, CR intake and the appropriate checkpoint path.
