---
handoff_id: "HANDOFF-20260518T173351-meta-po-to-meta-dev-STORY-004-implementation"
from_agent: "meta-po"
to_agent: "meta-dev"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
batch_id: "W2-dev-after-STORY-003-contract"
story_id: "STORY-004"
wave_id: "W2"
task_type: "implementation"
status: "completed"
created_at: "2026-05-18T17:33:51+0800"
preferred_existing_agent:
  agent_name: "dev-kong"
  agent_id: "019e39fc-797a-72d0-84d9-95f91f067ab7"
  thread_id: "019e39fc-797a-72d0-84d9-95f91f067ab7"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  agent_path: ".agents/agents/meta-dev.md"
  tool_name: "send_input"
  agent_id: "019e39fc-797a-72d0-84d9-95f91f067ab7"
  agent_name: "dev-kong"
  thread_id: "019e39fc-797a-72d0-84d9-95f91f067ab7"
  spawned_at: ""
  resumed_at: "2026-05-18T17:37:47+0800"
  completed_at: "2026-05-18T17:49:39+0800"
  evidence: "resume_agent+send_input"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: STORY-004 Implementation

## Dispatch Status

This handoff is not execution evidence. The host must use platform subagent dispatch and then write back dispatch evidence.

Preferred dispatch:

1. `resume_agent` / `send_input` to existing `dev-kong` if recoverable:
   - `agent_id`: `019e39fc-797a-72d0-84d9-95f91f067ab7`
   - `thread_id`: `019e39fc-797a-72d0-84d9-95f91f067ab7`
2. If the existing thread cannot be resumed, `spawn_agent` a new `meta-dev` for the exact same `workflow_id + change_id + story_id + wave_id`.

After real dispatch, update this file:

- `status`
- `dispatch.mode=subagent`
- `dispatch.tool_name=resume_agent | send_input | spawn_agent`
- `dispatch.agent_id`
- `dispatch.agent_name`
- `dispatch.thread_id`
- `dispatch.resumed_at` or `dispatch.spawned_at`
- `dispatch.completed_at` when finished
- `dispatch.evidence`

## Required Inputs

Read these files:

- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md`
- `process/ARCHITECTURE-DECISION.md`
- `process/PLATFORM-INSTALL-SPEC.md`
- `process/stories/STORY-004-model-multi-device-batch-configuration-contract.md`
- `process/stories/STORY-004-model-multi-device-batch-configuration-contract-LLD.md`
- `process/checks/CP5-STORY-004-model-multi-device-batch-configuration-contract-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md`
- `process/checks/CP6-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-CODING-DONE.md`
- `schemas/atomic-op.schema.yaml`
- `docs/schema-field-reference.md`
- `docs/naming-convention.md`
- `atoms/fw/fw_config_interface.yaml`
- `atoms/fw/fw_verify_interface.yaml`
- `packages/ngfw_capacity_config.yaml`
- `packages/ngfw_verification.yaml`
- `scripts/validate_schema.py`
- `scripts/layout_check.py`

## Task

Implement STORY-004 only: create the multi-device batch configuration contract.

Required outcomes:

- Create exactly 10 batch configuration atoms, one per STORY-003 capacity domain.
- Create `packages/ngfw_batch_config.yaml` with exactly the 10 `fw_config_batch_<domain>` op_ids.
- Create `docs/batch-configuration-contract.md` covering inventory references, selectors, concurrency, idempotency, failure isolation, `partial_failed`, verification summary, sensitive boundaries, and no automatic rollback.
- Use STORY-001 schema v1.1 contract and STORY-003 frozen 10-domain contract as strong inputs.
- Keep the current repository CLI read-only: do not add execution commands or device connectivity.
- Do not copy `.input/` content, real inventory, real IPs, usernames, passwords, tokens, cookies, FTP credentials, request bodies, response bodies, or runtime logs.

## Unique Write Scope

Allowed product writes:

- `atoms/fw/fw_config_batch_interface.yaml`
- `atoms/fw/fw_config_batch_object.yaml`
- `atoms/fw/fw_config_batch_acl_policy.yaml`
- `atoms/fw/fw_config_batch_policy_route.yaml`
- `atoms/fw/fw_config_batch_static_route.yaml`
- `atoms/fw/fw_config_batch_nat.yaml`
- `atoms/fw/fw_config_batch_bandwidth.yaml`
- `atoms/fw/fw_config_batch_black_white_list.yaml`
- `atoms/fw/fw_config_batch_ssl_vpn.yaml`
- `atoms/fw/fw_config_batch_dynamic_routing.yaml`
- `packages/ngfw_batch_config.yaml`
- `docs/batch-configuration-contract.md`

Required process write after implementation:

- `process/checks/CP6-STORY-004-model-multi-device-batch-configuration-contract-CODING-DONE.md`

Forbidden writes:

- `.input/*`
- `delivery/*`
- `schemas/*`
- `src/*`
- `scripts/*` except reading existing scripts
- `README.md`
- `CHANGELOG.md`
- `packages/ngfw_installation.yaml`
- `packages/ngfw_capacity_config.yaml`
- `packages/ngfw_verification.yaml`
- `docs/schema-field-reference.md`
- `docs/error-codes.md`
- `docs/naming-convention.md`
- `docs/USER-MANUAL.md`
- `docs/engineer-handbook.md`
- `docs/test-case-template.yaml`
- `atoms/fw/fw_config_interface.yaml`
- `atoms/fw/fw_verify_interface.yaml`
- `atoms/fw/fw_config_object.yaml`
- `atoms/fw/fw_verify_object.yaml`
- any STORY-001 / STORY-002 / STORY-003 atom or package file
- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/stories/STORY-001*`
- `process/stories/STORY-002*`
- `process/stories/STORY-003*`
- `process/stories/STORY-005*`
- `process/stories/STORY-006*`
- any CP7 file

## Required Checks

Run the smallest relevant checks and record command, result, and key output in CP6:

- `uv run --python 3.11 python scripts/validate_schema.py` over the 10 new `atoms/fw/fw_config_batch_*.yaml` files.
- `uv run --python 3.11 python scripts/layout_check.py`
- `uv run atomic-ops validate --package ngfw_batch_config`
- 10-domain coverage review: every STORY-003 domain has exactly one matching `fw_config_batch_<domain>` atom.
- Package scope review: `ngfw_batch_config` has exactly 10 batch op_ids and no single-device STORY-003 op_ids.
- Batch contract review: every batch atom contains non-sensitive `device_inventory_ref`, `device_selector`, `batch_ref`, `batch.max_concurrency`, `config_domain`, `params`, `state_ref` or `session_ref`, and `idempotency_key`.
- Concurrency review: `batch.max_concurrency` default is 1, minimum 1, maximum 5; high-risk batch default is 1.
- Failure isolation review: every batch atom returns or documents `batch_status`, `per_device_results[]`, `failed_devices[]`, and `verification_summary_ref`; `batch_status` includes `partial_failed`.
- High-risk gate review: every batch atom has `risk.level=high`, `gate.required=true`, non-empty `gate.reason`, `approver_role`, and `evidence_required`.
- Sensitive-pattern review over the 10 atoms, package, and batch doc: no real IP, token, cookie, authorization header, FTP credential, raw secret, original default password, real inventory, request body, response body, or `.input/` copy.
- Boundary review: no batch executor, no network/device connection, no automatic rollback/revert/undo of successful devices, and no CLI real-device action commands.
- Diff/file-scope check confirming no forbidden path was changed.

If `atomic-ops validate --package ngfw_batch_config` cannot see uncommitted worktree files because of cache behavior, record the default-cache result and then run the same CLI command against a worktree-backed temporary cache, as STORY-003 CP6 did. Do not mark CP6 `PASS` if schema, layout, package resolution, domain coverage, sensitive scan, or file-scope checks fail.

## CP6 Output Requirements

Create `process/checks/CP6-STORY-004-model-multi-device-batch-configuration-contract-CODING-DONE.md` with:

- Entry Criteria
- Checklist
- Exit Criteria
- Deliverables
- Agent Dispatch Evidence from this handoff
- Changed files and TASK-ID mapping S004-T1..S004-T13
- STORY-001 batch/schema contract evidence
- STORY-003 10-domain contract evidence
- 10-domain batch coverage evidence
- Atom/package reference evidence
- Batch concurrency and idempotency evidence
- Failure isolation and `partial_failed` evidence
- High-risk gate coverage evidence
- Security/sensitive scan evidence
- Rollback strategy
- Conclusion: `PASS`, `FAIL`, or `BLOCKED`

Do not create CP7 and do not call meta-qa. CP7 starts only after meta-po reviews CP6 and moves the Story to verification.

## Completion Criteria

- STORY-004 product files are implemented within the write scope.
- Required checks are run or explicitly blocked with reasons.
- CP6 file exists and includes dispatch evidence.
- The implementation establishes stable batch contract facts for downstream STORY-005 and STORY-006.
