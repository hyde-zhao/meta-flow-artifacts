---
handoff_id: "HANDOFF-20260518T152840-meta-po-to-meta-dev-STORY-006-lld"
from_agent: "meta-po"
to_agent: "meta-dev"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
story_id: "STORY-006"
wave_id: "W3"
batch_id: "CR-003-LLD-BATCH"
status: "completed"
created_at: "2026-05-18T15:28:40+0800"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  agent_path: ".codex/agents/meta-dev.toml"
  tool_name: "spawn_agent"
  agent_id: "019e39fc-f3e8-7381-beae-85bef34273ab"
  agent_name: "dev-xu"
  thread_id: "019e39fc-f3e8-7381-beae-85bef34273ab"
  spawned_at: "2026-05-18T15:28:40+0800"
  resumed_at: ""
  completed_at: "2026-05-18T15:34:50+0800"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: STORY-006 LLD

## Spawn Request

Host spawned one `meta-dev` sub agent for this handoff.

Assigned nickname: `dev-xu`

Reuse key: `role=meta-dev + workflow_id=atomic-ops-ngfw-install-config-chain + change_id=CR-003 + story_id=STORY-006 + wave_id=W3`

## Task

Write only the LLD for `STORY-006`. Do not modify README, docs, CHANGELOG, code, or delivery artifacts; this is design only.

Output file:

- `process/stories/STORY-006-update-user-facing-docs-and-release-guidance-LLD.md`

## Required Inputs

- `process/STATE.md`
- `process/changes/CR-003.md`
- `process/HLD.md`
- `process/ARCHITECTURE-DECISION.md`
- `process/PLATFORM-INSTALL-SPEC.md`
- `process/STORY-BACKLOG.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `process/stories/STORY-006-update-user-facing-docs-and-release-guidance.md`
- `process/stories/STORY-001-freeze-schema-v11-contract-and-field-docs-LLD.md`
- `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms-LLD.md`
- `process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms-LLD.md`
- `skills/lld-designer/templates/STORY-LLD-TEMPLATE.md`

## LLD Contract

- Use the 14 visible sections from `skills/lld-designer/templates/STORY-LLD-TEMPLATE.md`.
- Set frontmatter `confirmed: false`.
- Cover documentation update boundaries, command consistency, uv examples, sensitive information policy, read-only CLI boundary, and op_id reference consistency.
- Stop after LLD output. Implementation remains blocked until all target Story LLDs, CP5 auto prechecks, and CP5 batch manual confirmation are complete.
