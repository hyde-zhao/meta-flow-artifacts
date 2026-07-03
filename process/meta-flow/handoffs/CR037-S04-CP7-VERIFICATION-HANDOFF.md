---
handoff_id: "H-CR037-S04-CP7-VERIFICATION"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
codex_agent_name: "qa-cao"
agent_id: "019f25e2-d7d2-73d2-a3b3-8a7f19e14e31"
dispatch_id: "D-CR037-META-QA-CP7-S04"
status: "dispatched"
created_at: "2026-07-03T10:49:04+08:00"
stage: "story-verification"
cr_id: "CR-037"
story_id: "CR037-S04"
context_ref: "process/context/stories/CR037-S04.CP7.verify-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
---

# CR037-S04 CP7 Verification Handoff

## Objective

Verify ledger compaction CLI, retention policy, archive/index/backup behavior, marker compatibility, and safety boundaries.

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
