---
handoff_id: H-CR037-S06-CP6-IMPLEMENTATION
cr_id: CR-037
story_id: CR037-S06
from_role: host-orchestrator
to_role: meta-dev
stage: story-execution
status: handoff-created
created_at: 2026-07-03T14:50:00+08:00
context_ref: process/context/stories/CR037-S06.CP6.work-packet.json
dispatch_id: D-CR037-META-DEV-CP6-S06
canonical_role: meta-dev
codex_agent_name: dev-yang
capsule_first: true
read_profile: compact
---

# CR037-S06 CP6 Implementation Handoff

## Objective

Implement CR037-S06: `PROJECT-SCALE.yaml`, `ROADMAP.yaml`, `MILESTONES.yaml`, and the corresponding project object readers/checkers. Keep `PROJECT.current.json` refs-only and keep project scale as a gate profile bias recommendation only.

## Required Context

Read the work packet first:

- `process/context/stories/CR037-S06.CP6.work-packet.json`

The packet contains Story acceptance criteria, Feature design refs, LLD, dependency input from S05 CP7, allowed write paths, forbidden write paths, and verification plan.

## Implementation Focus

- Add project scale validation with `lite | standard | full`.
- Validate `gate_profile_bias.default_profile` against known profile IDs or allow empty; do not create a second gate profile namespace.
- Reject authorization-like scale fields such as `auto_approve`, `skip_gate`, `runtime_authorization`, `publish_authorization`, or direct gate policy mutation.
- Add roadmap and milestone baseline schema validation, status enums, duplicate ID detection, and cross-reference validation.
- Extend project snapshot loading from `PROJECT.current.json` refs.
- Update `meta-flow project check` to include scale, roadmap, and milestone validation.
- Create process-side baseline objects under `process/project/`.

## Allowed Writes

- `meta_flow/project/__init__.py`
- `meta_flow/project/scale.py`
- `meta_flow/project/roadmap.py`
- `meta_flow/project/state.py`
- `meta_flow/cli.py`
- `process/project/PROJECT-SCALE.yaml`
- `process/project/ROADMAP.yaml`
- `process/project/MILESTONES.yaml`
- `process/project/PROJECT.current.json`
- `tests/test_cr037_project_scale_roadmap.py`
- `process/stories/STORY-CR037-S06-project-scale-and-roadmap-objects-IMPLEMENTATION.md`
- `process/returns/CR037-S06.CP6.return.json`
- `process/evidence/CR037-S06.CP6.index.json`
- `process/checks/CP6-CR037-S06-CODING-DONE.result.json`
- `process/checks/CP6-CR037-S06-CODING-DONE.result.summary.md`

## Forbidden Writes

- `process/policies/GATE-PROFILES.json`
- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- `.env`
- any credential, secret, token, cookie, or private-key path

## Required Verification

Run at minimum:

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr037_project_scale_roadmap.py tests/test_cr037_project_current.py tests/test_state_v2.py tests/test_cr_lifecycle.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow project check --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root . --mode enforce
git diff -- process/policies/GATE-PROFILES.json process/quant-lab docs
git diff --check -- meta_flow/project/__init__.py meta_flow/project/scale.py meta_flow/project/roadmap.py meta_flow/project/state.py meta_flow/cli.py tests/test_cr037_project_scale_roadmap.py process/project/PROJECT-SCALE.yaml process/project/ROADMAP.yaml process/project/MILESTONES.yaml process/project/PROJECT.current.json
```

## Output Requirements

Write CP6 implementation summary, return packet, evidence index, CP6 result JSON, and CP6 summary only under the allowed process paths. Do not update Story status, ledgers, `STATE.current.json`, or CP7 files; host-orchestrator will integrate the CP6 return.
