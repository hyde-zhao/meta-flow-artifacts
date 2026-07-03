---
handoff_id: "HANDOFF-20260519T121233-meta-po-to-meta-pm-cr004-requirements"
from_agent: "meta-po"
to_agent: "meta-pm"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-004"
batch_id: "CR-004-REQUIREMENTS"
story_id: ""
wave_id: "requirement-clarification"
task_type: "cr004-requirements-and-use-cases"
status: "handoff-created"
created_at: "2026-05-19T12:12:33+0800"
reuse_decision:
  exact_reusable_agent_found: false
  decision: "spawn-new-meta-pm"
  rationale: "No active meta-pm entry exists for workflow_id=atomic-ops-ngfw-install-config-chain, change_id=CR-004, wave_id=requirement-clarification."
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  agent_path: ".agents/agents/meta-pm.md"
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: "handoff-only; current tool surface did not expose spawn_agent/resume_agent/send_input"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: CR-004 Requirements And Use Cases

## Dispatch Status

This file is a handoff request, not execution evidence. The host must spawn a `meta-pm` subagent, then write back real dispatch evidence in the frontmatter.

Required dispatch evidence after execution:

- `dispatch.mode=subagent`
- `dispatch.tool_name=spawn_agent | resume_agent | send_input | resume_agent+send_input`
- `dispatch.agent_id`
- `dispatch.agent_name`
- `dispatch.thread_id`
- `dispatch.spawned_at` or `dispatch.resumed_at`
- `dispatch.completed_at`
- `dispatch.evidence`

## Current Facts

- Workflow is currently reopened from `delivered` by `CR-004`.
- `process/changes/CR-004.md` records a high-impact change: add a public adapter profile layer so external script generators no longer need hidden `.input/capacity` implementation details.
- User preauthorized automatic progression for CR-004 artificial approval gates in this turn. See `checkpoints/CR-004-AUTO-PROCEED-AUTHORIZATION.md`.
- The authorization does not waive subagent dispatch evidence for CP6 / CP7.
- Current meta-po did not edit requirements, use cases, HLD, implementation files, or docs in this handoff step.

## Required Inputs

Read before writing:

- `process/STATE.md`
- `process/changes/CR-004.md`
- `process/USE-CASES.md`
- `process/REQUIREMENTS.md`
- `process/CLARIFICATION-LOG.md`
- `README.md`
- `docs/engineer-handbook.md`
- `docs/schema-field-reference.md`
- `docs/batch-configuration-contract.md`
- `atoms/fw/fw_config_policy_route.yaml`
- `packages/ngfw_capacity_config.yaml`

Do not load full historical handoff transcripts unless a cited fact is missing from the files above.

## Task

As `meta-pm`, perform only requirement-clarification work for CR-004:

1. Incrementally update `process/USE-CASES.md` with a scenario for script generators consuming public adapter profiles without `.input/capacity`.
2. Incrementally update `process/REQUIREMENTS.md` with adapter profile / spec requirements and explicit non-goals.
3. Update `process/CLARIFICATION-LOG.md` with the user feedback and the preauthorized approval context.
4. Produce or update CP1 / CP2 CR-004 automatic precheck evidence if the local checkpoint structure requires it.
5. Do not edit HLD, ADR, Story backlog, LLD, implementation files, product docs, or delivery artifacts.

## Expected Output

- Updated `process/USE-CASES.md`
- Updated `process/REQUIREMENTS.md`
- Updated `process/CLARIFICATION-LOG.md`
- Optional CR-004 CP1 / CP2 check files under `process/checks/`
- A short completion summary for meta-po, including changed files and unresolved requirement questions.

## Forbidden

- Do not modify `.input/`.
- Do not create adapter schema, adapter profile, atom refs, or docs; those belong to later `meta-se` / `meta-dev` / `meta-doc` steps.
- Do not mark CR-004 complete.
- Do not claim CP6 / CP7 readiness.
