---
handoff_id: "H-CR037-S01-CP7-VERIFICATION"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
codex_agent_name: "qa-he"
agent_id: "019f25bf-96a7-79c2-a9eb-d8ec1ec40bd3"
dispatch_id: "D-CR037-META-QA-CP7-S01"
status: "completed"
created_at: "2026-07-03T00:00:00+08:00"
completed_at: "2026-07-03T10:15:40+08:00"
stage: "story-verification"
cr_id: "CR-037"
story_id: "CR037-S01"
context_ref: "process/context/stories/CR037-S01.CP7.verify-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
---

# CR037-S01 CP7 Verification Handoff

## Objective

Verify `CR037-S01 current-state schema and budgets` after CP6 PASS.

## Must Read

- `process/context/stories/CR037-S01.CP7.verify-packet.json`
- `process/returns/CR037-S01.CP6.return.json`
- `process/evidence/CR037-S01.CP6.index.json`
- `process/checks/CP6-CR037-S01-CODING-DONE.result.json`
- `meta_flow/state/current.py`
- `tests/test_state_v2.py`

## Read If Needed

- `process/stories/STORY-CR037-S01-current-state-schema-and-budgets-IMPLEMENTATION.md`

## Allowed Writes

- `process/returns/CR037-S01.CP7.return.json`
- `process/evidence/CR037-S01.CP7.index.json`
- `process/checks/CP7-CR037-S01-VERIFICATION-DONE.result.json`
- `process/checks/CP7-CR037-S01-VERIFICATION-DONE.result.summary.md`

## Forbidden Writes

- `process/quant-lab/**`
- `meta_flow/**`
- `tests/**`
- `delivery/**`
- `docs/**`
- credential, secret, token, cookie or private-key paths

## Authorization Boundary

No implementation edits, runtime, production write, publish, live trading, real external calls, credential reads, or quant-lab release repository writes are authorized.
