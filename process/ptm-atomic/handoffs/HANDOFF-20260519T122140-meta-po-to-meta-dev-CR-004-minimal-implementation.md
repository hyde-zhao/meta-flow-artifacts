---
handoff_id: "HANDOFF-20260519T122140-meta-po-to-meta-dev-CR-004-minimal-implementation"
from_agent: "meta-po"
to_agent: "meta-dev"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-004"
batch_id: "CR-004-MINIMAL-DEV"
story_id: "CR004-MINIMAL"
wave_id: "CR-004"
task_type: "minimal-implementation"
status: "completed"
created_at: "2026-05-19T12:21:40+0800"
reuse_decision:
  exact_reusable_agent_found: false
  decision: "host-spawn-agent"
  rationale: "Host orchestrator used the Codex spawn_agent tool to create a real meta-dev subagent for CR-004 minimal implementation after the meta-po handoff requested dispatch."
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  agent_path: ".agents/agents/meta-dev.md"
  tool_name: "spawn_agent"
  agent_id: "019e3e7a-24c2-7ad1-8707-5e66eae0f69b"
  agent_name: "dev-you"
  thread_id: "019e3e7a-24c2-7ad1-8707-5e66eae0f69b"
  spawned_at: "2026-05-19T12:27:00+0800"
  resumed_at: ""
  completed_at: "2026-05-19T12:28:16+0800"
  evidence: "host-spawn-agent-completed"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: CR-004 Minimal Implementation

## Scope

Perform only the CR-004 minimal implementation requested by the user. Do not replay the full delivered workflow.

## Required Inputs

- `process/changes/CR-004.md`
- `schemas/adapter-profile.schema.yaml`
- `adapters/ngfw/default.yaml`
- `atoms/fw/fw_config_policy_route.yaml`
- `schemas/atomic-op.schema.yaml`
- `scripts/validate_adapter_profiles.py`
- `scripts/security_gate_check.py`
- `README.md`
- `docs/engineer-handbook.md`
- `docs/schema-field-reference.md`
- `docs/batch-configuration-contract.md`
- `CHANGELOG.md`

## Tasks

1. Ensure `schemas/adapter-profile.schema.yaml` defines a public, non-sensitive adapter profile contract.
2. Ensure `adapters/ngfw/default.yaml` covers at least `fw_config_policy_route`.
3. Add `adapter_contract_ref: adapters/ngfw/default.yaml#/operations/fw_config_policy_route` to `atoms/fw/fw_config_policy_route.yaml`.
4. Add `docs/API-ADAPTER-SPEC.md` and minimal README/docs updates.
5. Extend validation/security checks so adapter profiles cannot contain runtime sensitive information, including fields or values for `secret`, `host`, `token`, `cookie`, `password`, authorization material, private keys, production response samples, or `.input/capacity` dependencies.
6. Create a CR-004 CP6 check file under `process/checks/` with implementation evidence and command results.

## Constraints

- Do not modify `.input/`.
- Do not add a runner or any device connection behavior.
- Keep `atomic-ops` CLI read-only.
- Do not revert unrelated working tree changes.
