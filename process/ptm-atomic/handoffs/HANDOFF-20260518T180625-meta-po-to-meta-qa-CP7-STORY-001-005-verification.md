---
handoff_id: "HANDOFF-20260518T180625-meta-po-to-meta-qa-CP7-STORY-001-005-verification"
from_agent: "meta-po"
to_agent: "meta-qa"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
batch_id: "CP7-STORY-001-005"
story_id: "STORY-001,STORY-002,STORY-003,STORY-004,STORY-005"
wave_id: "W1-W2"
task_type: "verification-batch"
status: "completed"
created_at: "2026-05-18T18:06:25+0800"
preferred_existing_agent:
  agent_name: ""
  agent_id: ""
  thread_id: ""
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-qa"
  agent_path: ".agents/agents/meta-qa.md"
  tool_name: "spawn_agent"
  agent_id: "019e3a91-a7c7-7920-814d-74c955d0ad70"
  agent_name: "qa-yan"
  thread_id: "019e3a91-a7c7-7920-814d-74c955d0ad70"
  spawned_at: "2026-05-18T18:11:14+0800"
  resumed_at: ""
  completed_at: "2026-05-18T18:20:21+0800"
  evidence: "host-spawned-meta-qa-for-CP7-STORY-001-005"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: CP7 Verification Batch for STORY-001..005

## Dispatch Status

This handoff is not execution evidence. The host must use platform subagent dispatch and then write back dispatch evidence.

Preferred dispatch:

1. Spawn a new `meta-qa` for `role=meta-qa + workflow_id=atomic-ops-ngfw-install-config-chain + change_id=CR-003 + batch_id=CP7-STORY-001-005`.
2. Suggested nickname: `qa-he`.
3. Do not reuse the earlier HLD review QA thread unless the host can prove it is recoverable and appropriate for this exact CP7 batch key. The existing HLD review task is closed and has a different task scope.

After real dispatch, update this file:

- `status`
- `dispatch.mode=subagent`
- `dispatch.tool_name=spawn_agent | resume_agent | send_input`
- `dispatch.agent_id`
- `dispatch.agent_name`
- `dispatch.thread_id`
- `dispatch.spawned_at` or `dispatch.resumed_at`
- `dispatch.completed_at` when finished
- `dispatch.evidence`

## Verification Scope

Verify these five Story implementations and produce CP7 evidence for each:

- `STORY-001`: Freeze schema v1.1 contract and field docs
- `STORY-002`: Model NGFW install init login guard atoms
- `STORY-003`: Model capacity ten domain config and verification atoms
- `STORY-004`: Model multi-device batch configuration contract
- `STORY-005`: Add read-only security gate and validation checks

`STORY-006` is not in scope. It remains blocked until all five upstream CP7 files exist and pass.

## Required Inputs

Read these files before verification:

- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `process/HLD.md`
- `process/ARCHITECTURE-DECISION.md`
- `process/PLATFORM-INSTALL-SPEC.md`
- `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md`
- `process/stories/STORY-001-freeze-schema-v11-contract-and-field-docs.md`
- `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms.md`
- `process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms.md`
- `process/stories/STORY-004-model-multi-device-batch-configuration-contract.md`
- `process/stories/STORY-005-add-read-only-security-gate-and-validation-checks.md`
- `process/stories/STORY-001-freeze-schema-v11-contract-and-field-docs-LLD.md`
- `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms-LLD.md`
- `process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms-LLD.md`
- `process/stories/STORY-004-model-multi-device-batch-configuration-contract-LLD.md`
- `process/stories/STORY-005-add-read-only-security-gate-and-validation-checks-LLD.md`
- `process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md`
- `process/checks/CP6-STORY-002-model-ngfw-install-init-login-guard-atoms-CODING-DONE.md`
- `process/checks/CP6-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-CODING-DONE.md`
- `process/checks/CP6-STORY-004-model-multi-device-batch-configuration-contract-CODING-DONE.md`
- `process/checks/CP6-STORY-005-add-read-only-security-gate-and-validation-checks-CODING-DONE.md`
- `schemas/atomic-op.schema.yaml`
- `docs/schema-field-reference.md`
- `docs/error-codes.md`
- `docs/naming-convention.md`
- `docs/batch-configuration-contract.md`
- `atoms/fw/*.yaml`
- `packages/*.yaml`
- `scripts/validate_schema.py`
- `scripts/layout_check.py`
- `scripts/security_gate_check.py`
- `src/atomic_ops/cli.py`
- `src/atomic_ops/commands/*.py`

## Required Process Writes

Allowed process writes:

- `process/TEST-STRATEGY.md`
- `process/VERIFICATION-REPORT.md`
- `process/checks/CP7-STORY-001-freeze-schema-v11-contract-and-field-docs-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-002-model-ngfw-install-init-login-guard-atoms-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-004-model-multi-device-batch-configuration-contract-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-005-add-read-only-security-gate-and-validation-checks-VERIFICATION-DONE.md`

Forbidden writes:

- Product files under `atoms/`, `packages/`, `schemas/`, `src/`, `scripts/`, `docs/`, `README.md`, `CHANGELOG.md`
- `.input/*`
- `delivery/*`
- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `process/stories/*`
- `checkpoints/*`
- any CP6 file
- any STORY-006 implementation or CP6 file

State updates after CP7 are owned by meta-po.

## Verification Tasks

1. Create or update `process/TEST-STRATEGY.md` before CP7 results. Use ISTQB / ISO 25010 style coverage, but keep it specific to this repo and the five Story implementations.
2. Run repository-level static validation:
   - `uv run --python 3.11 python scripts/validate_schema.py atoms`
   - `uv run --python 3.11 python scripts/layout_check.py`
   - `uv run --python 3.11 python scripts/security_gate_check.py`
   - `uv run atomic-ops --help`
3. Verify package and op_id availability for implemented packages:
   - `ngfw_installation`
   - `ngfw_capacity_config`
   - `ngfw_verification`
   - `ngfw_batch_config`
4. Verify STORY-001:
   - schema v1.1 contract is valid
   - field reference, error codes, and naming docs align with schema and implemented atoms
   - CP6 dispatch evidence exists
5. Verify STORY-002:
   - exactly 5 install/init/login/health atoms exist and validate
   - `ngfw_installation` references resolve
   - high-risk gate and sensitive boundaries hold
   - CP6 dispatch evidence exists
6. Verify STORY-003:
   - 10 capacity domains exist
   - 20 config/verify op_ids exist
   - `ngfw_capacity_config` and `ngfw_verification` references resolve
   - D-004 package scope is respected
   - CP6 dispatch evidence exists
7. Verify STORY-004:
   - exactly 10 batch config atoms exist
   - `ngfw_batch_config` references resolve
   - concurrency, idempotency, failure isolation, and high-risk gates hold
   - CP6 dispatch evidence exists
8. Verify STORY-005:
   - `scripts/security_gate_check.py` default scan returns `0`
   - fixture evidence covers exit codes `31`, `32`, `33`
   - output redacts sensitive values
   - CLI command boundary still has no `run`, `execute`, `apply`, or `configure`
   - CP6 dispatch evidence exists
9. Write one CP7 result per Story using checkpoint-manager structure:
   - Entry Criteria
   - Checklist
   - Exit Criteria
   - Deliverables
   - Agent Dispatch Evidence from this handoff
   - Commands run and key outputs
   - Defects / risks
   - Conclusion: `PASS`, `FAIL`, `WAIVED`, or `BLOCKED`
10. Write `process/VERIFICATION-REPORT.md` summarizing the batch, cross-Story integration, residual risks, and whether STORY-006 remains blocked.

## CP7 Evidence Requirements

Each CP7 file must include Agent Dispatch Evidence with:

- `dispatch.mode=subagent`
- `dispatch.tool_name`
- `dispatch.agent_id` or `dispatch.thread_id`
- `dispatch.agent_name`
- `dispatch.spawned_at` or `dispatch.resumed_at`
- `dispatch.completed_at`
- `dispatch.evidence`

If the host has not filled this handoff dispatch block with real subagent evidence, CP7 conclusion must be `BLOCKED` or `FAIL`; do not mark verification as passed from handoff-only evidence.

## Completion Criteria

- `process/TEST-STRATEGY.md` exists or is updated for this verification batch.
- `process/VERIFICATION-REPORT.md` summarizes verification results and residual risks.
- All five CP7 result files exist.
- Each CP7 result includes valid meta-qa Agent Dispatch Evidence.
- No product files are modified.
- No STORY-006 implementation is started.
