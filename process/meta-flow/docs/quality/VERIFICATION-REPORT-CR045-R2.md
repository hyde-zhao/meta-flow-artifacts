# CR-045 CP7 Independent Reverification Report — R2

- Decision: `NEEDS_REWORK`
- Story: `CR045-S1`
- Validation mode: `mixed`
- Context: `process/context/CP7-CR045.context.json`
- Rework return: `process/returns/CR045-S1.CP6-R2.return.json`
- Rework evidence: `process/evidence/CR045-S1.CP6-R2.index.json`
- QA dispatch: `DISPATCH-CR045-CP7-REVERIFY-R2-QA`
- Checked at: `2026-07-11T11:45:43Z`

CP2 and CP5 remain recovery gates approved after the historical CP6. R2 does not reinterpret or backdate either approval.

## Scope

R2 independently reverified the two original HIGH findings, their positive and negative matrices, historical CP6 inline-fallback compatibility, R1 real-QA-dispatch compatibility, route/applicability/state/lifecycle behavior, focused tests, complete repository regression, static cleanliness, and delivery guardrail status. No implementation, test, delivery, state, ledger, CR-frontmatter, or R1 QA artifact was modified by this QA lane.

## Original finding closure

| Finding | R2 implementation | Independent evidence | Result |
|---|---|---|---|
| `CR045-F-001` — reason-only waiver passed | `route_plan.py` now requires both non-empty reason and ref | reason-only blocks; ref-only blocks; neither blocks; both fields produce explicit CP7 `WAIVED` | `CLOSED` |
| `CR045-F-002` — dispatch lookup checked only ID | `cp_result.py` now validates event semantics for applicable CP6/CP7 | wrong role/checkpoint/status/type/mode and incomplete fallback reject; historical valid fallback and real QA spawn pass | `CLOSED` |

## Verification object inventory

| Object | Method | Result |
|---|---|---|
| `meta_flow/policies/route_plan.py` | diff review, waiver matrix, route tests | `PASS` |
| `meta_flow/checks/cp_result.py` | diff review, dispatch matrix, real ledger compatibility | `PASS` |
| `tests/test_route_plan.py` | independent execution and semantic coverage review | `PASS` |
| `tests/test_cp_result_event_ledger.py` | independent execution and semantic coverage review | `PASS` |
| Historical CP6 result/dispatch | consistency check against `ADE-CR045-INLINE-CP6` | `PASS` |
| R1 CP7 result/dispatch | consistency check against `DISPATCH-CR045-CP7-QA` | `PASS` |
| Route/applicability/state/lifecycle | CLI contract checks | `PASS` |
| Repository regression | `uv run pytest` | `315 passed` |
| State transition post-PASS semantics | live CP7 PASS probe against stale `needs_rework` state | `FAIL` |
| Delivery guardrail | cache-sensitive guardrail | `NON_CR_TRANSIENT_FAIL` |

## Negative matrices

### Waiver matrix

| Reason | Ref | Expected | Result |
|---:|---:|---|---|
| absent | absent | route `BLOCKED`, CP7 not waived | `PASS` |
| present | absent | route `BLOCKED`, CP7 not waived | `PASS` |
| absent | present | route `BLOCKED`, CP7 not waived | `PASS` |
| present | present | route `PASS`, CP7 explicitly `WAIVED` with ref | `PASS` |

### Dispatch matrix

| Case | Expected | Result |
|---|---|---|
| missing dispatch ID | reject | `PASS` |
| wrong canonical role | reject | `PASS` |
| wrong checkpoint | reject | `PASS` |
| failed or running status | reject | `PASS` |
| `dispatch_not_required` for applicable CP6/CP7 | reject | `PASS` |
| incomplete inline fallback | reject | `PASS` |
| real dispatch with incompatible inline/not-required mode | reject | `PASS` |
| historical valid CP6 inline fallback | accept | `PASS` |
| valid CP7 real spawn dispatch | accept | `PASS` |
| CP6/CP7 `N/A` result | no dispatch fields required | `PASS` |

## Commands and results

| Command | Result |
|---|---|
| `git diff --check` | `PASS` |
| `uv run pytest tests/test_route_plan.py tests/test_cp_result_event_ledger.py` | `60 passed` |
| focused six-file suite | `120 passed` |
| `uv run pytest` | `315 passed` |
| CR-045 route check | `PASS` with expected auto-derived-verification warning |
| historical CP6 result consistency | `PASS` |
| R1 CP7 result consistency | `PASS` |
| dispatch/checkpoint/gate ledgers | `PASS` |
| applicability aggregate | `PASS` |
| State v2 audit | `PASS` |
| R1 CP7 state transition | `PASS` |
| CR lifecycle / gate profile | `PASS` with the same expected route warning |

The R2 dev return reported 59 tests for the two-file local suite; independent execution collected and passed 60. This is a non-blocking evidence-count correction, not a product defect.

## Semantic/security/compatibility review

- No runtime, credential, production-write, external-write, publication, or trading authority was added.
- The waiver repair is minimal and preserves explicit `N/A` behavior.
- Dispatch semantic checks are limited to applicable CP6/CP7 results and preserve `N/A` compatibility.
- The historical approved CP6 inline fallback remains valid.
- R1's real `meta-qa` spawn remains valid.
- Invalid event types cannot satisfy an applicable CP6/CP7 result merely by sharing an ID.
- The two original HIGH findings are closed, but a new HIGH state-transition finding was identified during final consistency validation.

## New finding

### CR045-F-003-R2 — HIGH — CP7 PASS accepts stale `needs_rework` stop state

The current machine state legitimately retains `next_action.stop_reason=needs_rework` from R1. Running:

```text
uv run meta-flow check state-transition --route-plan process/checks/CP0-CR045.route-plan.json --checkpoint CP7 --decision PASS --project-root .
```

returns `State Transition Check: OK` instead of requiring the next route target `pending_gate=CP8`. In `meta_flow/checks/state_transition.py`, `_has_valid_stop_reason()` treats any allowed reason other than `required_human_gate` and `delivered` as valid without checking whether it contradicts the current checkpoint decision or expected transition. Therefore a CP7 `PASS` can remain in stale `needs_rework`, `needs_design_clarification`, or another unrelated stop state and still satisfy consistency. This permits mutually contradictory machine facts and can prevent deterministic post-approval progression. Owner: `meta-dev`; route: `rework_same_story`.

## Non-CR repository hygiene

Delivery guardrail still detects ignored `__pycache__` and `.pyc` artifacts generated by Python tests. `PYTHONDONTWRITEBYTECODE=1` prevents new bytecode but does not remove caches already present. `git status --short` lists only the four intended R2 implementation/test modifications. This remains a Host Orchestrator CP8 hygiene action, not a CR-045 implementation defect and not a reason to keep the two HIGH findings open.

## Decision

`NEEDS_REWORK`. `CR045-F-001` and `CR045-F-002` are independently closed, but `CR045-F-003-R2` must be fixed before CP8. A pass-like automatic CP result must not accept a stale failure/rework stop reason as satisfying the expected transition to the next required gate. After rework, independently test PASS with stale `needs_rework`, `needs_design_clarification`, and `blocked` reasons, plus legitimate required-gate and authorization/workflow-health stop cases.
