---
handoff_id: "H-CR037-S03-CP7-VERIFICATION"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
codex_agent_name: "qa-hua"
agent_id: "019f2616-eaa3-7060-b7e1-7f53ac4ff709"
dispatch_id: "D-CR037-META-QA-CP7-S03"
status: "dispatched"
created_at: "2026-07-03T11:42:17+08:00"
stage: "story-verification"
cr_id: "CR-037"
story_id: "CR037-S03"
context_ref: "process/context/stories/CR037-S03.CP7.verify-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
---

# CR037-S03 CP7 Verification Handoff

## Objective

Verify the Agent / Skill current-state write contract synchronization.

## Must Read

- `process/context/stories/CR037-S03.CP7.verify-packet.json`
- `process/returns/CR037-S03.CP6.return.json`
- `process/evidence/CR037-S03.CP6.index.json`
- `process/checks/CP6-CR037-S03-CODING-DONE.result.json`

## Allowed Writes

- `process/returns/CR037-S03.CP7.return.json`
- `process/evidence/CR037-S03.CP7.index.json`
- `process/checks/CP7-CR037-S03-VERIFICATION-DONE.result.json`
- `process/checks/CP7-CR037-S03-VERIFICATION-DONE.result.summary.md`

## Forbidden Writes

- `meta_flow/**`
- `tests/**`
- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- credential, secret, token, cookie or private-key paths

## Authorization Boundary

No implementation edits, runtime, production write, publish, live trading, real external calls, credential reads, or quant-lab release repository writes are authorized.
