---
handoff_id: "HO-CR045-CP7-REWORK-META-DEV"
workflow_id: "meta-flow-self-dev"
change_id: "CR-045"
story_id: "CR045-S1"
stage: "CP7-REWORK"
from_role: "host-orchestrator"
to_role: "meta-dev"
codex_agent_name: "meta-dev-debugger"
reasoning_profile: "debugger"
status: "ready-for-dispatch"
context_ref: "process/context/CP7-CR045.context.json"
finding_ref: "process/docs/quality/FIXES-CR045.md"
created_at: "2026-07-11T11:36:03Z"
---

# CR-045 CP7 Rework Handoff

## Objective

Fix exactly `CR045-F-001` and `CR045-F-002`, add negative regression coverage, and return the Story to independent CP7 reverification. Do not broaden CR-045.

## Required Inputs

- `process/context/CP7-CR045.context.json`
- `process/docs/quality/REVIEW-CR045.md`
- `process/docs/quality/FIXES-CR045.md`
- `process/checks/CP7-CR045.result.json`
- `process/checkpoints/CP5-CR045-RECOVERY-DESIGN.md`
- relevant implementation and tests referenced by the findings

## Required Fixes

1. Route-plan verification waiver:
   - applicable CP7 can be `WAIVED` only when both non-empty `verification_waiver_reason` and non-empty `verification_waiver_ref` exist;
   - reason-only or ref-only input must BLOCK route derivation;
   - `N/A` semantics remain unchanged.
2. CP6/CP7 dispatch reference semantics:
   - reject `dispatch_not_required` for applicable CP6/CP7;
   - require expected `canonical_role` (`meta-dev` for CP6, `meta-qa` for CP7);
   - require matching checkpoint;
   - require terminal successful status;
   - accept a real dispatch only with platform/tool execution evidence;
   - accept inline fallback only when all approval fields required by the current contract exist;
   - reject unrelated, failed, running, incomplete, wrong-role, wrong-checkpoint and wrong-mode events.
3. Preserve compatibility for valid historical CR-045 CP6 inline fallback and the current real CP7 dispatch.

## Required Tests

- missing waiver ref
- missing waiver reason
- valid reason + ref
- wrong dispatch role
- wrong checkpoint
- failed/running dispatch
- `dispatch_not_required`
- incomplete inline fallback
- valid real spawn dispatch
- valid approved inline fallback
- existing route-plan, applicability, state-transition, lifecycle, event ledger and full repository regression

## Allowed Writes

- `meta_flow/policies/route_plan.py`
- `meta_flow/checks/cp_result.py`
- `meta_flow/state/event_ledger.py` only if required by the minimal fix
- `tests/test_route_plan.py`
- `tests/test_cp_result_event_ledger.py`
- `process/stories/CR045-S1-IMPLEMENTATION-R2.md`
- `process/returns/CR045-S1.CP6-R2.return.json`
- `process/evidence/CR045-S1.CP6-R2.index.json`

Do not modify CP7 QA reports/results, CR frontmatter, state, ledgers, release files, unrelated code, or historical CP6 evidence.

## Return Contract

Report exact changed files, commands, results, remaining risks, and whether QA can reverify. If either fix requires a broader schema/architecture change, stop with `NEEDS_DESIGN_CLARIFICATION` instead of expanding scope.
