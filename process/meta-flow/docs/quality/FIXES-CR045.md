# CR-045 CP7 Rework Input

Status: `required`
Source decision: `NEEDS_REWORK`

CP2 and CP5 were recovery gates approved after historical CP6; rework must preserve that audit ordering.

## Required fixes

| Finding | Owner | Required change | Minimum regression |
|---|---|---|---|
| CR045-F-001 | meta-dev | In route derivation, block CP7 waiver unless both `verification_waiver_reason` and `verification_waiver_ref` are present and non-empty. Keep `N/A` distinct. | negative missing-ref test; valid ref test; profile upgrade and phase-order tests |
| CR045-F-002 | meta-dev | For applicable CP6/CP7, resolve each dispatch ref to a semantically valid event. Reject `dispatch_not_required`; require expected canonical role/checkpoint, successful status, and valid real-dispatch or approved inline-fallback fields. | wrong role, wrong checkpoint, failed status, dispatch-not-required, incomplete fallback, valid spawn, valid approved fallback |

## Reverification

Rerun the focused CR-045 tests, `uv run pytest`, route/result/applicability/event/state/CR/gate checks, and delivery guardrails. Host integration must update state before `CP7 result-check --check-consistency` is expected to pass.
