---
handoff_id: "H-CR037-S05-CP6-IMPLEMENTATION"
from_role: "host-orchestrator"
to_role: "meta-dev"
canonical_role: "meta-dev"
codex_agent_name: "dev-xu"
agent_id: "019f26a3-f31b-7bd0-8590-5ac220627def"
dispatch_id: "D-CR037-META-DEV-CP6-S05"
status: "handoff-created"
created_at: "2026-07-03T14:30:00+08:00"
stage: "story-execution"
cr_id: "CR-037"
story_id: "CR037-S05"
context_ref: "process/context/stories/CR037-S05.CP6.work-packet.json"
capsule_first: true
read_profile: "compact"
can_ask_user: false
---

# CR037-S05 CP6 Implementation Handoff

## Objective

Implement `process/project/` scaffold support and refs-only `PROJECT.current.json` governance state without embedding project governance fields into `STATE.current.json`.

## Must Read

- `process/context/stories/CR037-S05.CP6.work-packet.json`
- `process/stories/STORY-CR037-S05-project-scaffold-and-project-current.md`

## Read If Needed

- `process/stories/STORY-CR037-S05-project-scaffold-and-project-current-LLD.md`
- `process/docs/features/project-state-governance/DESIGN.md`
- `process/docs/features/project-state-governance/TEST-PLAN.md`
- `process/docs/features/project-state-governance/TASKS.md`
- `meta_flow/workspace/routing.py`
- `meta_flow/state/current.py`
- `meta_flow/cli.py`

## Allowed Writes

- `meta_flow/project/__init__.py`
- `meta_flow/project/state.py`
- `meta_flow/project/scaffold.py`
- `meta_flow/workspace/routing.py`
- `meta_flow/state/current.py`
- `meta_flow/cli.py`
- `process/project/PROJECT.current.json`
- `tests/test_cr037_project_current.py`
- `process/stories/STORY-CR037-S05-project-scaffold-and-project-current-IMPLEMENTATION.md`
- `process/returns/CR037-S05.CP6.return.json`
- `process/evidence/CR037-S05.CP6.index.json`
- `process/checks/CP6-CR037-S05-CODING-DONE.result.json`
- `process/checks/CP6-CR037-S05-CODING-DONE.result.summary.md`

## Forbidden Writes

- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- `.env`
- credential, secret, token, cookie or private-key paths

## Authorization Boundary

No runtime, production write, publish, live trading, real external calls, credential reads, quant-lab release repository writes, or real ledger apply are authorized.

`process/project/PROJECT.current.json` may be created only as the CR037-S05 scaffold baseline and must remain refs-only. `STATE.current.json` may be updated only through the controlled writer and only for `project_state_ref`.

## Verification Expectations

- `meta-flow context check-story-packet --packet process/context/stories/CR037-S05.CP6.work-packet.json --project-root .`
- Target tests for project current / scaffold.
- State regression tests covering `project_state_ref` and `routing_ref`.
- `meta-flow state check --project-root . --mode enforce`.
- `git diff -- process/quant-lab docs` must be empty.
