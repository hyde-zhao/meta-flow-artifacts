---
status: "completed"
version: "1.0"
story_id: "ST-EI-001"
story_slug: "gate-chronology"
feature_id: "FEAT-EI-CORE"
implementation_type: "code-rework"
source_story: "process/stories/STORY-ST-EI-001-gate-chronology.md"
source_design_evidence: "process/stories/STORY-ST-EI-001-gate-chronology-LLD.md"
supersedes_implementation_ref: "process/stories/STORY-ST-EI-001-gate-chronology-IMPLEMENTATION.md"
created_by: "host-orchestrator"
created_at: "2026-07-12T04:33:49Z"
---

# Implementation R2: ST-EI-001 rolling CP7 transition correction

## Rework trigger

During pre-CP7 route validation, the new CP5 allowance correctly represented CP6/CP7 work in progress, but the existing transition guard still required a preliminary Story-level CP7 to open global CP8. That would conflate a rolling Story checkpoint with a CR-level final gate.

## Corrected contract

`_is_automatic_phase_in_progress` now allows a CP7 `PASS` or `PASS_WITH_RISK` to advance the dependency graph only when all of the following are true:

1. state remains `story-execution`;
2. `active_story` identifies a next Story;
3. no future human gate is pre-populated;
4. next action is not an await-user/blocked/done action.

The final CP7 remains required to open CP8 because it must clear `active_story`; existing pending-CP8 tests remain intact.

## Changed files and local verification

| File | Change | Verification |
|---|---|---|
| `meta_flow/checks/state_transition.py` | Extend automatic-progress guard to rolling CP7 without weakening final CP8 behavior. | `uv run pytest tests/test_state_transition.py -q` |
| `tests/test_state_transition.py` | Add `test_cp7_pass_like_decision_accepts_next_story_before_final_cp8`. | 28 passed, 42 subtests passed |

No design delta is required: this is the LLD §2/§7 phase/gate separation rule made complete. No external runtime, credential, publish, commit/push, historical mutation, or quant-lab business source access occurred.

## Supersession

- Previous CP6 attempt: `process/checks/CP6-CR046-ST-EI-001-gate-chronology-CODING-DONE.result.json`
- Current CP6 attempt: `process/checks/CP6-CR046-ST-EI-001-gate-chronology-CODING-DONE-R2.result.json`
- The original attempt remains intact; R2 is the current implementation evidence for CP7.
