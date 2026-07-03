---
handoff_id: "H-CR037-S04-CP7-VERIFICATION-R2"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
codex_agent_name: "qa-lv"
agent_id: "019f2617-1c99-73b1-8b49-989d8af925a2"
dispatch_id: "D-CR037-META-QA-CP7-S04-R2"
status: "dispatched"
created_at: "2026-07-03T11:42:17+08:00"
stage: "story-verification"
cr_id: "CR-037"
story_id: "CR037-S04"
context_ref: "process/context/stories/CR037-S04.CP7.verify-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
retry_of: "D-CR037-META-QA-CP7-S04"
---

# CR037-S04 CP7 Verification Handoff Retry

## Objective

Retry CP7 verification for CR037-S04 after the previous QA dispatch hit a usage limit before writing outputs.

## Must Read

- `process/context/stories/CR037-S04.CP7.verify-packet.json`
- `process/returns/CR037-S04.CP6.return.json`
- `process/evidence/CR037-S04.CP6.index.json`
- `process/checks/CP6-CR037-S04-CODING-DONE.result.json`

## Allowed Writes

- `process/returns/CR037-S04.CP7.return.json`
- `process/evidence/CR037-S04.CP7.index.json`
- `process/checks/CP7-CR037-S04-VERIFICATION-DONE.result.json`
- `process/checks/CP7-CR037-S04-VERIFICATION-DONE.result.summary.md`

## Forbidden Writes

- `meta_flow/**`
- `tests/**`
- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- credential, secret, token, cookie or private-key paths

## Authorization Boundary

No implementation edits, real ledger apply, runtime, production write, publish, live trading, real external calls, credential reads, or quant-lab release repository writes are authorized.
