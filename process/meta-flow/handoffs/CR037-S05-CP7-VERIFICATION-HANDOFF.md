---
handoff_id: H-CR037-S05-CP7-VERIFICATION
cr_id: CR-037
story_id: CR037-S05
from_role: host-orchestrator
to_role: meta-qa
stage: story-verification
status: handoff-created
created_at: 2026-07-03T14:35:09+08:00
context_ref: process/context/stories/CR037-S05.CP7.verify-packet.json
dispatch_id: D-CR037-META-QA-CP7-S05
canonical_role: meta-qa
codex_agent_name: qa-he
capsule_first: true
read_profile: compact
---

# CR037-S05 CP7 Verification Handoff

## Objective

Verify CR037-S05 after CP6 implementation. The verifier must confirm the project scaffold and `PROJECT.current.json` implementation satisfies the approved Story contract, does not create a second project-governance state mechanism, and does not write outside the allowed CP7 evidence/result paths.

## Required Context

Read the capsule first:

- `process/context/stories/CR037-S05.CP7.verify-packet.json`

The capsule contains the required Story, CR summary, read policy, feature refs, acceptance criteria, dependency inputs, and verification plan. Read full LLD or implementation files only when the capsule, return packet, or evidence index is insufficient, and record any full-document expansion in `process/state/READ-EXPANSION-LEDGER.ndjson`.

## Verification Focus

- `process/project/PROJECT.current.json` is refs-only, budgeted, allowlisted, and checked through `meta-flow project check`.
- `STATE.current.json` only references project governance through `project_state_ref`; it does not embed roadmap, milestone, scale, capability, or migration details.
- `routing_ref` remains a required and budgeted current-state key after the post-review refinement.
- Workspace scaffold coverage includes `process/project` without breaking existing symlink routing.
- Project scaffold CLI defaults to dry-run and apply does not overwrite conflicting existing `PROJECT.current.json`.
- CP6 return, evidence index, and result JSON are structurally valid.
- `git diff -- process/quant-lab docs` remains empty.

## Allowed Writes

- `process/returns/CR037-S05.CP7.return.json`
- `process/evidence/CR037-S05.CP7.index.json`
- `process/checks/CP7-CR037-S05-VERIFICATION-DONE.result.json`
- `process/checks/CP7-CR037-S05-VERIFICATION-DONE.result.summary.md`

## Forbidden Writes

- `meta_flow/**`
- `tests/**`
- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- `.env`
- any credential, secret, token, cookie, or private-key path

## Required Verification Commands

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr037_project_current.py tests/test_state_v2.py tests/test_cr_lifecycle.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow project check --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root . --mode enforce
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow cp result-check --result process/checks/CP6-CR037-S05-CODING-DONE.result.json --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow story return-check --packet process/context/stories/CR037-S05.CP6.work-packet.json --return process/returns/CR037-S05.CP6.return.json --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow story evidence-check --index process/evidence/CR037-S05.CP6.index.json --project-root .
git diff -- process/quant-lab docs
```

## Expected Outcome

Write CP7 evidence and result with `PASS_WITH_RISK`, `PASS`, `NEEDS_REWORK`, or `BLOCKED` according to verification evidence. Do not update Story status, ledgers, `STATE.current.json`, source files, tests, or long-lived docs; host-orchestrator will integrate the CP7 return.
