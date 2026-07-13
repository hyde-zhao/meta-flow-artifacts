# CR-045 CP7 Test Report

Decision: `NEEDS_REWORK`
Dispatch: `DISPATCH-CR045-CP7-QA`

CP2 and CP5 were recovery gates approved after the historical CP6; their approvals are not treated as pre-implementation authorization.

## Executed tests and checks

| Command | Result | Coverage |
|---|---|---|
| `git diff --check` | PASS | whitespace/static cleanliness |
| `uv run pytest tests/test_route_plan.py tests/test_gate_policy.py tests/test_cp_result_event_ledger.py tests/test_state_transition.py tests/test_cr_lifecycle.py tests/test_ask_user.py` | PASS, 111 tests | route, profiles, result/ledger, transition, lifecycle, replay |
| `uv run pytest` | PASS, 306 tests | complete repository regression |
| `uv run meta-flow route check --from-cr process/changes/CR-045.md --project-root .` | PASS with expected warning | frontmatter bridge and stale artifact |
| CP2/CP5/CP6 `cp result-check --check-consistency --mode silent` | PASS | result schema, ledger refs, existing state consistency |
| applicability/checkpoint/dispatch/gate ledger silent checks | PASS | applicability and event schemas |
| state audit, CR lifecycle, gate profile | PASS | state and lifecycle consumers |
| CP5 human-gate replay and validation | PASS | replay output contract |
| `uv run python scripts/check_delivery_guardrails.py` | FAIL (unrelated/transient) | ignored Python caches generated during test execution |

## Negative and semantic coverage

The existing suite covers stale route artifacts, profile upgrades, `N/A`, waiver records in CP results, structured dispatch event required fields, applicability drift, silent output, transition routing, lifecycle checks, and replay. Independent semantic probes exposed missing negative coverage:

- no test rejects route-plan `WAIVED` when `verification_waiver_reason` exists but `verification_waiver_ref` is absent;
- no test rejects an applicable CP6/CP7 result whose `dispatch_refs` resolves to `dispatch_not_required`, an unrelated role/checkpoint, or a non-completed dispatch.

An independent temporary-ledger probe confirmed the second gap: a `dispatch_not_required` event for canonical role `meta-dev` with ID `FAKE-CP7` produced `_validate_dispatch_refs(...) == []` when used as a CP7 dispatch reference.

## Coverage gaps and defects

| ID | Severity | Test gap / failure | Required action |
|---|---|---|---|
| CR045-F-001 | HIGH | reason-only waiver produces route PASS and removes CP7 from stages | require approval ref and add negative test |
| CR045-F-002 | HIGH | dispatch consistency verifies only ID existence | validate semantic dispatch contract and add negative tests |
| CR045-O-001 | INFO | current CP5 transition check cannot pass until host integrates CP7 | host advances state and reruns consistency after result integration |

Passing automated tests do not override these negative semantic failures. Full details: `process/docs/quality/VERIFICATION-REPORT-CR045.md` and `process/docs/quality/REVIEW-CR045.md`.
