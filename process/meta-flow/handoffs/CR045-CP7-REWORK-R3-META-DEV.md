---
handoff_id: "HO-CR045-CP7-REWORK-R3-META-DEV"
workflow_id: "meta-flow-self-dev"
change_id: "CR-045"
story_id: "CR045-S1"
stage: "CP7-REWORK-R3"
from_role: "host-orchestrator"
to_role: "meta-dev"
codex_agent_name: "meta-dev-debugger"
reasoning_profile: "debugger"
status: "ready-for-dispatch"
context_ref: "process/context/CP7-CR045.context.json"
finding_ref: "process/docs/quality/FIXES-CR045-R2.md"
created_at: "2026-07-11T11:55:43Z"
---

# CR-045 CP7 Rework R3

Fix only `CR045-F-003-R2`.

Required behavior:

- pass-like CP decisions must not be accepted with stale failure stop reasons such as `needs_rework`, `needs_design_clarification`, or `blocked`;
- for CR-045 CP7 PASS/PASS_WITH_RISK, the route must reach pending required CP8 unless a decision-compatible authorization/workflow-health stop applies;
- failure decisions must continue to accept their matching failure stop reason;
- preserve route-plan N/A/WAIVED and post-approval behavior;
- add decision × stop-reason negative and positive tests.

Allowed implementation writes:

- `meta_flow/checks/state_transition.py`
- `tests/test_state_transition.py`
- `tests/test_cp_result_event_ledger.py` only if CP result integration coverage is required
- `process/stories/CR045-S1-IMPLEMENTATION-R3.md`
- `process/returns/CR045-S1.CP6-R3.return.json`
- `process/evidence/CR045-S1.CP6-R3.index.json`

Do not modify QA artifacts, CR/state/ledgers, route plan, release files, or earlier evidence. Run local, focused and full regression. Use dispatch ID `DISPATCH-CR045-CP7-REWORK-R3-DEV`.
