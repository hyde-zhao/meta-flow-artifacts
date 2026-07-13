# CR-045 CP7 Independent Verification Report

- Decision: `NEEDS_REWORK`
- Story: `CR045-S1`
- Validation mode: `mixed` (static review + executable contract tests + repository regression)
- Context: `process/context/CP7-CR045.context.json`
- Dispatch: `DISPATCH-CR045-CP7-QA` (`meta-qa-critical`, `spawn_agent`, agent `/root/qa_critical_cr045`)
- Checked at: `2026-07-11T11:32:01Z`

CP2 and CP5 were recovery gates approved after the historical CP6. This report treats them only as ex-post recovery confirmations and does not reinterpret either gate as a pre-implementation approval.

## Scope and non-scope

The verification covers route-plan derivation and frontmatter bridging, profile upgrades, checkpoint phase order, stale route artifacts, `N/A` versus `WAIVED`, dispatch evidence, silent checks, applicability aggregation, human-gate replay, state transitions, CR lifecycle integration, delivery guardrails, security/permission boundaries, focused tests, and full regression. It does not authorize runtime access, credentials, external writes, publication, production changes, or repository push. It does not modify implementation code, tests, state, ledgers, CR frontmatter, or delivery assets.

## Verification object inventory

| Object | Verification method | Result |
|---|---|---|
| `meta_flow/policies/route_plan.py` | semantic review, positive/negative route tests, live negative probe | `FAIL` |
| `meta_flow/checks/cp_result.py` | semantic review, result/dispatch reverse-lookup tests and ledger checks | `FAIL` |
| `meta_flow/state/event_ledger.py` | schema review and focused tests | `PASS_WITH_FINDING` |
| `meta_flow/checks/state_transition.py` | focused tests and current CP5 transition check | `PASS_WITH_INTEGRATION_NOTE` |
| `meta_flow/workflow/cr_lifecycle.py` | focused tests and `meta-flow cr check` | `PASS` |
| `meta_flow/ask_user.py` | replay test and live CP5 replay | `PASS` |
| Route/applicability artifacts | route check and applicability check | `PASS` |
| Delivery guardrail | repository guardrail command | `FAIL_UNRELATED_TRANSIENT` |
| Repository regression | 306-test full suite | `PASS` |

## Traceability matrix

| Promise / contract | Implementation | Evidence | Result | Risk |
|---|---|---|---|---|
| `CR045-EV-ROUTE`: deterministic route contract | `route_plan.py` | 111 focused tests; route check; stale-artifact tests | `PASS` | none |
| `CR045-EV-CP-LEDGER`: result and ledger contract | `cp_result.py`, `event_ledger.py` | result checks; event checks; semantic negative review | `FAIL` | `CR045-F-002` |
| `CR045-EV-REGRESSION`: repository regression | repository | `uv run pytest` => 306 passed | `PASS` | none |
| Approved CP5 contract: strict `N/A` / `WAIVED` distinction | `route_plan.py` | live negative probe accepts reason-only waiver | `FAIL` | `CR045-F-001` |
| Approved CP5 contract: current state/lifecycle consumers | `state_transition.py`, `cr_lifecycle.py` | state audit and lifecycle pass; CP5 transition fails while CP7 is in progress | `PASS_WITH_INTEGRATION_NOTE` | `CR045-O-001` |
| Human-gate replay | `ask_user.py` | 6 tests plus CP5 replay/check | `PASS` | none |

## Design-contract verification

| Contract | Source | Blocking | Result |
|---|---|---:|---|
| Applicable verification may be `WAIVED` only with explicit risk-acceptance evidence | CP5 result; checkpoint-manager waiver contract | yes | `FAIL`: reason-only waiver is accepted as route `PASS` and omits `waiver_ref` |
| CP6/CP7 dispatch evidence must prove a real role-appropriate dispatch or approved inline fallback | CP5 result; checkpoint-manager dispatch contract | yes | `FAIL`: consistency lookup checks only that an ID exists in the dispatch ledger |
| `N/A` means checkpoint does not apply and is distinct from `WAIVED` | CP5 result; route plan | yes | `PASS` for ordinary `N/A`; waiver authorization half is incomplete |
| Recovery approvals must not be backdated | CP2/CP5 results | yes | `PASS`: reports preserve historical ordering |
| No new runtime/credential/publication authority | CR summary and capsule | yes | `PASS` |

## Layered execution plan and results

| Layer | Command / method | Result |
|---|---|---|
| Static | `git diff --check` | `PASS` |
| Focused unit/contract | `uv run pytest tests/test_route_plan.py tests/test_gate_policy.py tests/test_cp_result_event_ledger.py tests/test_state_transition.py tests/test_cr_lifecycle.py tests/test_ask_user.py` | `111 passed` |
| Full regression | `uv run pytest` | `306 passed` |
| Route artifact | `uv run meta-flow route check --from-cr process/changes/CR-045.md --project-root .` | `PASS` with expected auto-derivation warning |
| CP results | CP2, CP5, CP6 `result-check --check-consistency --mode silent` | all `PASS` |
| Applicability | `meta-flow cp applicability-check ... --mode silent` | `PASS` |
| Event ledgers | checkpoint, dispatch, gate silent checks | all `PASS` |
| State/CR/gate | state audit, CR lifecycle, gate profile | all `PASS` |
| Human gate | CP5 replay plus human-gate validation | `PASS` |
| Guardrail | `uv run python scripts/check_delivery_guardrails.py` | failed only on ignored `__pycache__`/`.pyc` generated by Python test execution; not attributed to CR-045 source behavior |

## Findings and remaining risks

### CR045-F-001 — HIGH — unapproved verification waiver can produce a passing route

`derive_route_plan()` checks only `verification_waiver_reason`; `verification_waiver_ref` is optional. A negative probe with new implementation, `has_new_verification=false`, and a free-text reason but no approval reference returned route `decision=PASS` and CP7 `decision=WAIVED`. This conflicts with the waiver contract requiring explicit approval evidence and can remove CP7 from `stages` without an auditable approval reference. Owner: `meta-dev`. Route: `rework_same_story`.

### CR045-F-002 — HIGH — dispatch reference lookup proves ID existence, not valid CP7 execution

`_validate_dispatch_refs()` builds a set of all dispatch/event IDs and accepts a match without checking event type, canonical role, checkpoint, status, dispatch mode, or approval fields. A temporary-ledger probe inserted `FAKE-CP7` as `dispatch_not_required` for `meta-dev`; `_validate_dispatch_refs(..., {"dispatch_refs":["FAKE-CP7"]})` returned `[]`. Consequently a CP7 result can reference a not-required event, an unrelated role, or a failed dispatch and satisfy the consistency check. This violates the non-waivable real-dispatch requirement. Owner: `meta-dev`. Route: `rework_same_story`.

### CR045-O-001 — integration note, not classified as CR-045 implementation defect

During active CP7 dispatch, `meta-flow check state-transition --approved-gate CP5` reports that state has not yet advanced to pending CP8. This is expected to remain unresolved until the Host Orchestrator consumes the CP7 result. The current state also labels its stop reason `required_human_gate` while `pending_gate` is empty; the host should normalize this during integration before running CP7 consistency validation. Owner: `host-orchestrator`.

### Historical/transient repository-health observation

The delivery guardrail detected ignored Python cache files created by test execution. `git status --short` remained clean, and the failure is not evidence of a CR-045 source regression. It must not be merged into CR045-F-001/F-002 severity, though the host may clean/avoid bytecode before CP8 guardrail rerun.

## Stage decision

`NEEDS_REWORK`. Both HIGH findings affect machine-enforced governance and permit false evidence to pass. The recommended minimal rework is:

1. Require both a non-empty waiver reason and a valid `verification_waiver_ref` before route-plan CP7 can be `WAIVED`; add a negative test for missing ref.
2. Make CP6/CP7 dispatch consistency validate the referenced ledger event's allowed type/mode, canonical role, checkpoint, successful status, and inline-fallback approval fields; explicitly reject `dispatch_not_required` for applicable CP6/CP7 results and add negative tests.
3. Rerun the focused 111 tests, new negative tests, all 306+ repository tests, route/result/ledger checks, and the guardrail with bytecode generation disabled or caches removed by the host.
