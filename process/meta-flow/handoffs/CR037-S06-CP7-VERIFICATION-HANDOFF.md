---
handoff_id: H-CR037-S06-CP7-VERIFICATION
cr_id: CR-037
story_id: CR037-S06
from_role: host-orchestrator
to_role: meta-qa
stage: story-verification
status: completed
created_at: 2026-07-03T15:44:27+08:00
completed_at: 2026-07-03T15:47:03+08:00
context_ref: process/context/stories/CR037-S06.CP7.verify-packet.json
dispatch_id: D-CR037-META-QA-CP7-S06
canonical_role: meta-qa
codex_agent_name: host-orchestrator-inline-fallback
capsule_first: true
read_profile: compact
inline_fallback: true
---

# CR037-S06 CP7 Verification Handoff

## Objective

Verify CR037-S06 after CP6 implementation and user-authorized CP6 inline-fallback. The verifier must prove that project scale, roadmap, milestones, refs-only project current state, and project checker behavior satisfy the approved Story contract.

## Required Context

Read the capsule first:

- `process/context/stories/CR037-S06.CP7.verify-packet.json`

Read full LLD or implementation files only if the capsule, CP6 return packet, or evidence index is insufficient, and record any full-document expansion in `process/state/READ-EXPANSION-LEDGER.ndjson`.

## Verification Focus

- `PROJECT-SCALE.yaml` supports `lite / standard / full`, gate profile bias, reason, review cadence bias, and explicit `not_authorized` boundaries.
- `gate_profile_bias` is a recommendation only and does not write `process/policies/GATE-PROFILES.json`.
- `ROADMAP.yaml` and `MILESTONES.yaml` validate schema, status enums, duplicate IDs, and cross refs.
- `PROJECT.current.json` remains refs-only and links `scale_ref`, `roadmap_ref`, and `milestones_ref`.
- `meta-flow project check --project-root .` validates project current, scale, roadmap, and milestones together.
- `git diff -- process/policies/GATE-PROFILES.json process/quant-lab docs` remains empty.

## Allowed Writes

- `process/returns/CR037-S06.CP7.return.json`
- `process/evidence/CR037-S06.CP7.index.json`
- `process/checks/CP7-CR037-S06-VERIFICATION-DONE.result.json`
- `process/checks/CP7-CR037-S06-VERIFICATION-DONE.result.summary.md`

## Forbidden Writes

- `meta_flow/**`
- `tests/**`
- `process/policies/GATE-PROFILES.json`
- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- `.env`
- any credential, secret, token, cookie, or private-key path

## Required Verification Commands

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr037_project_scale_roadmap.py tests/test_cr037_project_current.py tests/test_state_v2.py tests/test_cr_lifecycle.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow project check --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root . --mode enforce
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow cp result-check --result process/checks/CP6-CR037-S06-CODING-DONE.result.json --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow story return-check --packet process/context/stories/CR037-S06.CP6.work-packet.json --return process/returns/CR037-S06.CP6.return.json --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow story evidence-check --index process/evidence/CR037-S06.CP6.index.json --project-root .
git diff -- process/policies/GATE-PROFILES.json process/quant-lab docs
```

## Completion Note

CP7 was completed by user-authorized host-orchestrator inline-fallback. This follows the approved fallback policy after repeated QA subagent interruptions and the S06 dev subagent producing code/tests but not process artifacts.
