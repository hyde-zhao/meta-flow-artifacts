---
handoff_id: HANDOFF-20260518T110000-meta-po-to-meta-se-solution-design
from_agent: meta-po
to_agent: meta-se
phase: solution-design
status: agent_completed
created_at: 2026-05-18T11:00:00+0800
dispatch:
  required: true
  mode: subagent
  platform: codex-api
  agent_role: meta-se
  requested_agent_type: meta-se
  actual_agent_type: default
  acting_role: meta-se
  agent_path: .codex/agents/meta-se.toml
  tool_name: spawn_agent
  agent_id: 019e3906-9216-7ad3-a198-1ad71d51fc00
  agent_name: Rawls
  thread_id: 019e3906-9216-7ad3-a198-1ad71d51fc00
  spawned_at: 2026-05-18T11:00:00+0800
  resumed_at: ""
  completed_at: 2026-05-18T11:01:06+0800
  evidence: "spawn_agent returned agent_id=019e3906-9216-7ad3-a198-1ad71d51fc00 nickname=Rawls; current API exposes only default/explorer/worker, so this is default acting as meta-se. Completed outputs: process/HLD.md and process/checks/CP3-HLD-CONSISTENCY.md; no ADR, Story backlog, development plan, or Story files were generated."
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: meta-po -> meta-se / solution-design

## Task

Produce the solution-design stage HLD and CP3 automatic precheck for the confirmed atomic-ops requirements baseline.

## Required Outputs

- `process/HLD.md`
- `process/checks/CP3-HLD-CONSISTENCY.md`

## Explicit Non-Outputs Before CP3 Approval

- `process/ARCHITECTURE-DECISION.md`
- `process/PLATFORM-INSTALL-SPEC.md`
- `process/STORY-BACKLOG.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `process/stories/STORY-*.md`

## Required Inputs

- `AGENTS.md`
- `.codex/agents/meta-se.toml`
- `.agents/skills/hld-designer/SKILL.md`
- `.agents/skills/checkpoint-manager/SKILL.md`
- `process/STATE.md`
- `process/USE-CASES.md`
- `process/REQUIREMENTS.md`
- `process/checks/CP2-REQUIREMENTS-BASELINE.md`
- `checkpoints/CP2-REQUIREMENTS-BASELINE.md`
- `README.md`

## Routing Constraints

- This is a production project; product deliverables must follow the README-native surface, not `delivery/`.
- `.input/` is reference-only and must not be copied directly as current deliverables.
- CP3 must pass automatic precheck and manual HLD review before Story planning or ADR output.
