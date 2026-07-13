# CR-045 CP7 R2 Test Report

Decision: `NEEDS_REWORK`
Dispatch: `DISPATCH-CR045-CP7-REVERIFY-R2-QA`

CP2 and CP5 were recovery gates approved after the historical CP6 and are not treated as pre-implementation approvals.

## Results

| Layer | Result | Evidence |
|---|---|---|
| Static diff | PASS | `git diff --check` |
| R2 local | PASS | 60/60 tests |
| CR-045 focused | PASS | 120/120 tests |
| Full repository | PASS | 315/315 tests |
| Waiver negative matrix | PASS | neither/reason-only/ref-only reject; both accept |
| Dispatch negative matrix | PASS | wrong role/checkpoint/status/type/mode and incomplete fallback reject |
| Compatibility | PASS | historical CP6 fallback and R1 real QA dispatch accept |
| Route/applicability/state/lifecycle | PASS | CLI checks |
| CP7 PASS against stale R1 rework state | FAIL | checker returns OK instead of requiring CP8 |
| Delivery guardrail | transient non-CR failure | ignored pre-existing test caches only |

## Closure evidence

- `CR045-F-001`: closed by explicit two-field waiver predicate plus reason-only/ref-only negative tests.
- `CR045-F-002`: closed by semantic dispatch validation plus negative tests and real ledger compatibility checks.

## Evidence discrepancy

The dev R2 evidence says the two-file local suite had 59 passes. Independent QA collected 60 and all passed. No test is missing; the recorded count was stale by one.

## Remaining coverage gaps

`CR045-F-003-R2` exposes a missing decision-sensitive transition matrix: pass-like results must reject stale failure/rework stop reasons. Add negative cases for `needs_rework`, `needs_design_clarification`, and `blocked`, and retain positive cases for the next required gate and explicitly legitimate non-failure stop reasons. Delivery cache cleanup remains a separate CP8 repository-hygiene prerequisite owned by the Host Orchestrator.
