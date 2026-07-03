---
handoff_id: "H-CR037-S03-CP6-IMPLEMENTATION"
from_role: "host-orchestrator"
to_role: "meta-dev"
canonical_role: "meta-dev"
codex_agent_name: "dev-kong"
agent_id: "019f25dd-1926-7b00-b26e-52c109a326ae"
dispatch_id: "D-CR037-META-DEV-CP6-S03"
status: "completed"
completed_at: "2026-07-03T11:42:17+08:00"
created_at: "2026-07-03T10:42:03+08:00"
stage: "story-execution"
cr_id: "CR-037"
story_id: "CR037-S03"
context_ref: "process/context/stories/CR037-S03.CP6.work-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
---

# CR037-S03 CP6 Implementation Handoff

## Objective

Sync the agent contract and state-router documentation with the S01/S02 current-state enforcement and controlled writer contract.

## Must Read

- `process/context/stories/CR037-S03.CP6.work-packet.json`
- `process/stories/STORY-CR037-S03-agent-contract-and-guardrail-sync.md`

## Allowed Writes

- `delivery/rules/AGENT-SKILL-CONTRACT.md`
- `.agents/skills/state-router/SKILL.md`
- `AGENTS.md`
- `delivery/rules/AGENTS.md`
- `process/stories/STORY-CR037-S03-agent-contract-and-guardrail-sync-IMPLEMENTATION.md`
- `process/returns/CR037-S03.CP6.return.json`
- `process/evidence/CR037-S03.CP6.index.json`
- `process/checks/CP6-CR037-S03-CODING-DONE.result.json`
- `process/checks/CP6-CR037-S03-CODING-DONE.result.summary.md`

## Forbidden Writes

- `process/quant-lab/**`
- `meta_flow/**`
- `tests/**`
- `docs/**`
- credential, secret, token, cookie or private-key paths

## Authorization Boundary

No runtime, production write, publish, live trading, real external calls, credential reads, quant-lab release repository writes, or current-state implementation edits are authorized.
