---
handoff_id: "HANDOFF-20260518T151444-meta-po-to-meta-dev-STORY-002-lld"
from_agent: "meta-po"
to_agent: "meta-dev"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
story_id: "STORY-002"
wave_id: "W1"
batch_id: "CR-003-LLD-BATCH"
status: "completed"
created_at: "2026-05-18T15:14:44+0800"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  agent_path: ".codex/agents/meta-dev.toml"
  tool_name: "spawn_agent"
  agent_id: "019e39f6-7759-7122-9216-871849ef788b"
  agent_name: "dev-zhang"
  thread_id: "019e39f6-7759-7122-9216-871849ef788b"
  spawned_at: "2026-05-18T15:22:03+0800"
  resumed_at: ""
  completed_at: "2026-05-18T15:26:58+0800"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: STORY-002 LLD

## Spawn Request

Host should spawn one `meta-dev` sub agent for this handoff.

Recommended nickname candidate: `dev-zhu`

Reuse key: `role=meta-dev + workflow_id=atomic-ops-ngfw-install-config-chain + change_id=CR-003 + story_id=STORY-002 + wave_id=W1`

## Task

Write only the LLD for `STORY-002`. Do not implement atom/package files, do not run CP6/CP7, and do not edit code or delivery artifacts.

Output file:

- `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms-LLD.md`

## Required Inputs

- `process/STATE.md`
- `process/changes/CR-003.md`
- `process/HLD.md`
- `process/ARCHITECTURE-DECISION.md`
- `process/STORY-BACKLOG.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms.md`
- `process/stories/STORY-001-freeze-schema-v11-contract-and-field-docs.md`
- `process/PLATFORM-INSTALL-SPEC.md`
- `skills/lld-designer/templates/STORY-LLD-TEMPLATE.md`

## LLD Contract

- Use the 14 visible sections from `skills/lld-designer/templates/STORY-LLD-TEMPLATE.md`.
- Set frontmatter `confirmed: false`.
- Treat `STORY-001` as an upstream contract dependency for LLD design only; implementation remains blocked until CP5 batch confirmation.
- Stop after LLD output. Implementation remains blocked until all target Story LLDs, CP5 auto prechecks, and CP5 batch manual confirmation are complete.
