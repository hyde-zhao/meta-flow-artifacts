---
handoff_id: "HANDOFF-20260518T171519-meta-po-to-meta-dev-STORY-002-implementation"
from_agent: "meta-po"
to_agent: "meta-dev"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
batch_id: "W1-parallel-dev-after-STORY-001-contract"
story_id: "STORY-002"
wave_id: "W1"
task_type: "implementation"
status: "completed"
created_at: "2026-05-18T17:15:19+0800"
preferred_existing_agent:
  agent_name: "dev-zhang"
  agent_id: "019e39f6-7759-7122-9216-871849ef788b"
  thread_id: "019e39f6-7759-7122-9216-871849ef788b"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  agent_path: ".agents/agents/meta-dev.md"
  tool_name: "send_input"
  agent_id: "019e39f6-7759-7122-9216-871849ef788b"
  agent_name: "dev-zhang"
  thread_id: "019e39f6-7759-7122-9216-871849ef788b"
  spawned_at: ""
  resumed_at: "2026-05-18T17:20:17+0800"
  completed_at: "2026-05-18T17:32:34+0800"
  evidence: "resume_agent+send_input"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: STORY-002 Implementation

## Dispatch Status

This handoff is not execution evidence. The host must use platform subagent dispatch and then write back dispatch evidence.

Preferred dispatch:

1. `resume_agent` / `send_input` to existing `dev-zhang` if recoverable:
   - `agent_id`: `019e39f6-7759-7122-9216-871849ef788b`
   - `thread_id`: `019e39f6-7759-7122-9216-871849ef788b`
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
- `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms.md`
- `process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms-LLD.md`
- `process/checks/CP5-STORY-002-model-ngfw-install-init-login-guard-atoms-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md`
- `schemas/atomic-op.schema.yaml`
- `docs/schema-field-reference.md`
- `docs/error-codes.md`
- `docs/naming-convention.md`
- `scripts/validate_schema.py`
- `scripts/layout_check.py`

## Task

Implement STORY-002 only: create the NGFW installation/init/login guard atoms and the installation package.

Required outcomes:

- Create 5 contract-only firewall atom YAML files for install, init, login, login-state guard, and health verification.
- Create `packages/ngfw_installation.yaml` as a package view that references the 5 new op_ids and does not copy atom bodies.
- Use STORY-001 schema v1.1 contract exactly as frozen by CP6: `schema_version: "1.1"`, non-sensitive refs, high-risk `risk` and `gate`, diagnostic refs, and no real executor semantics.
- Preserve CLI read-only boundary: do not add or imply real device execution commands.
- Do not copy `.input/ngfw-install` implementation, credentials, runtime logs, real IPs, FTP credentials, cookies, tokens, or response payloads.

## Unique Write Scope

Allowed product writes:

- `atoms/fw/fw_install_ngfw_image.yaml`
- `atoms/fw/fw_init_ngfw_minimal.yaml`
- `atoms/fw/fw_login_web_management.yaml`
- `atoms/fw/fw_check_login_state.yaml`
- `atoms/fw/fw_verify_ngfw_health.yaml`
- `packages/ngfw_installation.yaml`

Required process write after implementation:

- `process/checks/CP6-STORY-002-model-ngfw-install-init-login-guard-atoms-CODING-DONE.md`

Forbidden writes:

- `.input/*`
- `delivery/*`
- `schemas/*`
- `docs/*`
- `src/*`
- `scripts/*` except reading existing scripts
- `README.md`
- `CHANGELOG.md`
- `packages/ngfw_capacity_config.yaml`
- `packages/ngfw_verification.yaml`
- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/stories/STORY-001*`
- `process/stories/STORY-003*`
- `process/stories/STORY-004*`
- `process/stories/STORY-005*`
- `process/stories/STORY-006*`
- any CP7 file

## Required Checks

Run the smallest relevant checks and record command, result, and key output in CP6:

- `uv run --python 3.11 python scripts/validate_schema.py atoms/fw/fw_install_ngfw_image.yaml atoms/fw/fw_init_ngfw_minimal.yaml atoms/fw/fw_login_web_management.yaml atoms/fw/fw_check_login_state.yaml atoms/fw/fw_verify_ngfw_health.yaml`
- `uv run --python 3.11 python scripts/layout_check.py`
- `uv run atomic-ops validate --package ngfw_installation`
- High-risk gate review over the 5 new atoms: every new atom must have `risk.level=high`, `gate.required=true`, non-empty `gate.reason`, `approver_role`, and `evidence_required`.
- Sensitive-pattern review over the 5 new atoms and package: no real IP, token, cookie, authorization header, FTP credential, raw secret, original default password, or response payload. `Ngfw@123` is allowed only as a password policy value.
- Boundary review: initialization is limited to the approved minimal actions; SSH/license must be absent; health verification failure must produce diagnostic/manual handling only and no automatic rollback.
- Diff/file-scope check confirming no forbidden path was changed.

If a required command cannot run because the supporting checker is not implemented yet, record the reason and use the LLD-approved manual/equivalent check where allowed. Do not mark CP6 `PASS` if schema, layout, package resolution, sensitive scan, or file-scope checks fail.

## CP6 Output Requirements

Create `process/checks/CP6-STORY-002-model-ngfw-install-init-login-guard-atoms-CODING-DONE.md` with:

- Entry Criteria
- Checklist
- Exit Criteria
- Deliverables
- Agent Dispatch Evidence from this handoff
- Changed files and TASK-ID mapping S002-T1..S002-T7
- STORY-001 contract evidence and schema version used
- Atom/package reference evidence
- High-risk gate coverage evidence
- Security/sensitive scan evidence
- Boundary review evidence for no real executor, no `.input/` copy, no SSH/license expansion, and no automatic rollback
- Rollback strategy
- Conclusion: `PASS`, `FAIL`, or `BLOCKED`

Do not create CP7 and do not call meta-qa. CP7 starts only after meta-po reviews CP6 and moves the Story to verification.

## Completion Criteria

- STORY-002 product files are implemented within the write scope.
- Required checks are run or explicitly blocked with reasons.
- CP6 file exists and includes dispatch evidence.
- The implementation establishes a stable installation package and atom set for downstream STORY-005 and STORY-006 planning.
