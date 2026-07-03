---
handoff_id: H-CR037-S07-CP6-IMPLEMENTATION
cr_id: CR-037
story_id: CR037-S07
from_role: host-orchestrator
to_role: meta-dev
stage: story-execution
status: completed
created_at: 2026-07-03T16:13:08+08:00
context_ref: process/context/stories/CR037-S07.CP6.work-packet.json
dispatch_id: D-CR037-META-DEV-CP6-S07
canonical_role: meta-dev
codex_agent_name: meta-dev
capsule_first: true
read_profile: compact
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "default"
  dispatch_trigger: "CR-037 S07 CP6 implementation after S05/S06 verified-with-risk"
  tool_name: "spawn_agent"
  agent_id: "019f270d-3bed-78d0-b0b0-3812d1b34854"
  agent_name: "dev-lv the 2nd"
  thread_id: "019f270d-3bed-78d0-b0b0-3812d1b34854"
  spawned_at: "2026-07-03T16:13:08+08:00"
  completed_at: "2026-07-03T17:27:30+08:00"
  evidence: "spawn_agent"
question_permission:
  can_ask_user: false
  mode: "queue-only"
  structured_choice_allowed: false
  allowed_question_scope: "Implementation blockers only; write a clarification item instead of asking user directly."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 gates, runtime authorization, credentials, security boundary changes, publish, live/trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  story_packet_ref: "process/context/stories/CR037-S07.CP6.work-packet.json"
  read_profile: "compact"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  allowed_reads:
    - "process/context/stories/CR037-S07.CP6.work-packet.json"
  must_read:
    - "process/context/stories/CR037-S07.CP6.work-packet.json"
  read_if_needed:
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
    - "meta_flow/design/product_governance.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/STORY-STATUS.md"
    - "process/changes/*.md"
    - "process/quant-lab/**"
    - "docs/**"
---

# CR037-S07 CP6 Implementation Handoff

## Objective

Implement CR037-S07: feature/capability registry and resolver. Extend the existing `meta_flow/design/feature_registry.py` implementation so `feature_refs` and `capability_refs` can be resolved through registry-backed refs rather than free strings.

## Required Context

Read the story packet first:

- `process/context/stories/CR037-S07.CP6.work-packet.json`

The packet contains the Story, LLD, Feature DESIGN / TEST-PLAN / TASKS, acceptance criteria, dependency inputs, allowed writes, forbidden writes, and verification plan. Read full upstream design documents only if the packet is insufficient or a conflict appears.

## Critical Instructions

- Do not write CP6 process artifacts. Host Orchestrator owns:
  - `process/stories/STORY-CR037-S07-feature-capability-registry-and-resolver-IMPLEMENTATION.md`
  - `process/returns/CR037-S07.CP6.return.json`
  - `process/evidence/CR037-S07.CP6.index.json`
  - `process/checks/CP6-CR037-S07-CODING-DONE.result.json`
  - `process/checks/CP6-CR037-S07-CODING-DONE.result.summary.md`
- Use `CAP-PG-*` placeholder capability IDs in fixtures and tests. Do not use quant-lab real capability IDs; real mapping belongs to S13.
- Do not read or write `process/quant-lab/**`.
- Do not modify `docs/**`, `delivery/**`, `process/returns/**`, `process/evidence/**`, `process/checks/**`, or `process/state/**`.

## Allowed Writes

- `meta_flow/design/feature_registry.py`
- `meta_flow/cli.py`
- `tests/test_feature_registry.py`

## Forbidden Writes

- `process/returns/**`
- `process/evidence/**`
- `process/checks/**`
- `process/state/**`
- `process/quant-lab/**`
- `process/docs/**`
- `docs/**`
- `delivery/**`
- `.env`
- any credential, secret, token, cookie, or private-key path

## Implementation Focus

- Feature registry v1/v2 compatibility.
- `CAPABILITY-REGISTRY.yaml` schema/checker.
- Resolver API with `resolved`, `unresolved`, `deprecated`, and `conflict`.
- Candidate report payload for unresolved refs without writing canonical registry.
- Synthetic consumer tests proving downstream code consumes resolver result rather than free strings.

## Required Verification Commands

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_feature_registry.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow feature check --project-root <fixture-root> --include-capabilities
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow feature resolve --project-root <fixture-root> --kind capability --ref CAP-PG-REGISTRY-REFS
git diff -- process/quant-lab docs delivery
git diff --check -- meta_flow/design/feature_registry.py meta_flow/cli.py tests/test_feature_registry.py
```

## Expected Return

Return with:

- changed source/test files only;
- commands run and results;
- any unresolved implementation blocker;
- confirmation that no CP6 process artifacts were written.

Host Orchestrator will run final validation and generate CP6 process evidence.
