---
handoff_id: "H-CR037-S04-CP6-IMPLEMENTATION"
from_role: "host-orchestrator"
to_role: "meta-dev"
canonical_role: "meta-dev"
codex_agent_name: "dev-he"
agent_id: "019f25d6-2d63-7b73-a809-f948122392f0"
dispatch_id: "D-CR037-META-DEV-CP6-S04"
status: "completed"
completed_at: "2026-07-03T10:49:04+08:00"
created_at: "2026-07-03T10:32:04+08:00"
stage: "story-execution"
cr_id: "CR-037"
story_id: "CR037-S04"
context_ref: "process/context/stories/CR037-S04.CP6.work-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
---

# CR037-S04 CP6 Implementation Handoff

## Objective

Implement independent ledger compaction policy and CLI without reusing `state compact`.

## Must Read

- `process/context/stories/CR037-S04.CP6.work-packet.json`
- `process/stories/STORY-CR037-S04-ledger-compaction-policy-and-cli.md`

## Read If Needed

- `process/stories/STORY-CR037-S04-ledger-compaction-policy-and-cli-LLD.md`
- `process/docs/features/ledger-compaction/DESIGN.md`
- `process/docs/features/ledger-compaction/TEST-PLAN.md`
- `process/docs/features/ledger-compaction/TASKS.md`

## Allowed Writes

- `meta_flow/state/ledger_compaction.py`
- `meta_flow/state/event_ledger.py`
- `meta_flow/cli.py`
- `meta_flow/state/current.py`
- `process/policies/LEDGER-RETENTION.yaml`
- `tests/test_cr037_ledger_compaction.py`
- `process/stories/STORY-CR037-S04-ledger-compaction-policy-and-cli-IMPLEMENTATION.md`
- `process/returns/CR037-S04.CP6.return.json`
- `process/evidence/CR037-S04.CP6.index.json`
- `process/checks/CP6-CR037-S04-CODING-DONE.result.json`
- `process/checks/CP6-CR037-S04-CODING-DONE.result.summary.md`

## Forbidden Writes

- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- credential, secret, token, cookie or private-key paths

## Authorization Boundary

No runtime, production write, publish, live trading, real external calls, credential reads, or quant-lab release repository writes are authorized. `ledger compact` apply behavior may be implemented and tested only against temporary fixtures.
