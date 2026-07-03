---
handoff_id: "H-CR037-S00-CP7-VERIFICATION"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
codex_agent_name: "qa-kong"
agent_id: "019f25ab-a83e-7e80-89e1-df44aff51085"
dispatch_id: "D-CR037-META-QA-CP7-S00"
status: "completed"
created_at: "2026-07-03T00:00:00+08:00"
completed_at: "2026-07-03T09:52:01+08:00"
stage: "story-verification"
cr_id: "CR-037"
story_id: "CR037-S00"
context_ref: "process/context/stories/CR037-S00.CP7.verify-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
---

# CR037-S00 CP7 Verification Handoff

## Objective

Verify `CR037-S00 second-system guardrail` after CP6 PASS. This is a static / review-only verification of the guardrail evidence package, not runtime validation.

## Must Read

- `process/context/stories/CR037-S00.CP7.verify-packet.json`
- `process/returns/CR037-S00.CP6.return.json`
- `process/evidence/CR037-S00.CP6.index.json`
- `process/checks/CP6-CR037-S00-CODING-DONE.result.json`

## Allowed Writes

- `process/returns/CR037-S00.CP7.return.json`
- `process/evidence/CR037-S00.CP7.index.json`
- `process/checks/CP7-CR037-S00-VERIFICATION-DONE.result.json`
- `process/checks/CP7-CR037-S00-VERIFICATION-DONE.result.summary.md`

## Forbidden Writes

- `process/quant-lab/**`
- `meta_flow/**`
- `delivery/**`
- `docs/**`
- `tests/**`
- credential, secret, token, account or private-key paths

## Authorization Boundary

No runtime, production write, publish, live trading, real external calls, credential reads, implementation edits, or quant-lab release repository writes are authorized.
