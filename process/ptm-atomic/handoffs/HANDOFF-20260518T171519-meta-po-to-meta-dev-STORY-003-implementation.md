---
handoff_id: "HANDOFF-20260518T171519-meta-po-to-meta-dev-STORY-003-implementation"
from_agent: "meta-po"
to_agent: "meta-dev"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
batch_id: "W1-parallel-dev-after-STORY-001-contract"
story_id: "STORY-003"
wave_id: "W1"
task_type: "implementation"
status: "completed"
created_at: "2026-05-18T17:15:19+0800"
preferred_existing_agent:
  agent_name: "dev-shi"
  agent_id: "019e39f6-806b-75f1-b5c0-0b859d019ede"
  thread_id: "019e39f6-806b-75f1-b5c0-0b859d019ede"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  agent_path: ".agents/agents/meta-dev.md"
  tool_name: "send_input"
  agent_id: "019e39f6-806b-75f1-b5c0-0b859d019ede"
  agent_name: "dev-shi"
  thread_id: "019e39f6-806b-75f1-b5c0-0b859d019ede"
  spawned_at: ""
  resumed_at: "2026-05-18T17:20:17+0800"
  completed_at: "2026-05-18T17:32:34+0800"
  evidence: "resume_agent+send_input"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: STORY-003 Implementation

## Dispatch Status

This handoff is not execution evidence. The host must use platform subagent dispatch and then write back dispatch evidence.

Preferred dispatch:

1. `resume_agent` / `send_input` to existing `dev-shi` if recoverable:
   - `agent_id`: `019e39f6-806b-75f1-b5c0-0b859d019ede`
   - `thread_id`: `019e39f6-806b-75f1-b5c0-0b859d019ede`
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
- `process/reviews/CP5-LLD-meta-se-architecture-review.md`
- `process/ARCHITECTURE-DECISION.md`
- `process/PLATFORM-INSTALL-SPEC.md`
- `process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms.md`
- `process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms-LLD.md`
- `process/checks/CP5-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md`
- `schemas/atomic-op.schema.yaml`
- `docs/schema-field-reference.md`
- `docs/naming-convention.md`
- `scripts/validate_schema.py`
- `scripts/layout_check.py`

## Task

Implement STORY-003 only: create the 10 capacity configuration atom pairs and the capacity/verification packages.

Required outcomes:

- Create exactly 10 configuration atoms and 10 verification atoms for the CP5-approved domains.
- Create `packages/ngfw_capacity_config.yaml` with exactly the 10 configuration op_ids.
- Create `packages/ngfw_verification.yaml` with exactly the 10 capacity verification op_ids. Per CP5 accepted D-004, do not add STORY-002 health-check op_ids to this package in this Story.
- Use STORY-001 schema v1.1 contract exactly as frozen by CP6: `schema_version: "1.1"`, non-sensitive refs, high-risk `risk` and `gate` for configuration atoms, verification refs, and no real executor semantics.
- Do not copy `.input/capacity` source, env files, logs, request bodies, response bodies, credentials, token/cookie material, or real device addresses.
- Do not add CLI real-device action commands.

## Unique Write Scope

Allowed product writes:

- `atoms/fw/fw_config_interface.yaml`
- `atoms/fw/fw_verify_interface.yaml`
- `atoms/fw/fw_config_object.yaml`
- `atoms/fw/fw_verify_object.yaml`
- `atoms/fw/fw_config_acl_policy.yaml`
- `atoms/fw/fw_verify_acl_policy.yaml`
- `atoms/fw/fw_config_policy_route.yaml`
- `atoms/fw/fw_verify_policy_route.yaml`
- `atoms/fw/fw_config_static_route.yaml`
- `atoms/fw/fw_verify_static_route.yaml`
- `atoms/fw/fw_config_nat.yaml`
- `atoms/fw/fw_verify_nat.yaml`
- `atoms/fw/fw_config_bandwidth.yaml`
- `atoms/fw/fw_verify_bandwidth.yaml`
- `atoms/fw/fw_config_black_white_list.yaml`
- `atoms/fw/fw_verify_black_white_list.yaml`
- `atoms/fw/fw_config_ssl_vpn.yaml`
- `atoms/fw/fw_verify_ssl_vpn.yaml`
- `atoms/fw/fw_config_dynamic_routing.yaml`
- `atoms/fw/fw_verify_dynamic_routing.yaml`
- `packages/ngfw_capacity_config.yaml`
- `packages/ngfw_verification.yaml`

Required process write after implementation:

- `process/checks/CP6-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-CODING-DONE.md`

Forbidden writes:

- `.input/*`
- `delivery/*`
- `schemas/*`
- `docs/*`
- `src/*`
- `scripts/*` except reading existing scripts
- `README.md`
- `CHANGELOG.md`
- `packages/ngfw_installation.yaml`
- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/stories/STORY-001*`
- `process/stories/STORY-002*`
- `process/stories/STORY-004*`
- `process/stories/STORY-005*`
- `process/stories/STORY-006*`
- any CP7 file

## Required Checks

Run the smallest relevant checks and record command, result, and key output in CP6:

- `uv run --python 3.11 python scripts/validate_schema.py` over the 20 new atom files.
- `uv run --python 3.11 python scripts/layout_check.py`
- `uv run atomic-ops validate --package ngfw_capacity_config`
- `uv run atomic-ops validate --package ngfw_verification`
- 10-domain coverage review: exactly 10 configuration domains, each with one `fw_config_<domain>` and one `fw_verify_<domain>` file.
- Package scope review: `ngfw_capacity_config` has exactly 10 configuration op_ids; `ngfw_verification` has exactly 10 capacity verification op_ids and no STORY-002 health-check op_id.
- High-risk gate review over the 10 configuration atoms: every configuration atom must have `risk.level=high`, `gate.required=true`, non-empty `gate.reason`, `approver_role`, and `evidence_required`.
- Verification failure boundary review: verification atoms produce diagnostic/manual handling references only and do not define automatic rollback/revert/undo actions.
- Sensitive-pattern review over the 20 atoms and 2 packages: no real IP, token, cookie, authorization header, FTP credential, raw secret, original default password, copied request/response payload, or `.input/capacity` source.
- Diff/file-scope check confirming no forbidden path was changed.

If a required command cannot run because the supporting checker is not implemented yet, record the reason and use the LLD-approved manual/equivalent check where allowed. Do not mark CP6 `PASS` if schema, layout, package resolution, sensitive scan, domain coverage, or file-scope checks fail.

## CP6 Output Requirements

Create `process/checks/CP6-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-CODING-DONE.md` with:

- Entry Criteria
- Checklist
- Exit Criteria
- Deliverables
- Agent Dispatch Evidence from this handoff
- Changed files and TASK-ID mapping S003-T1..S003-T13
- STORY-001 contract evidence and schema version used
- 10-domain coverage evidence
- Atom/package reference evidence
- High-risk gate coverage evidence
- Verification failure boundary evidence
- Security/sensitive scan evidence
- D-004 evidence that `ngfw_verification` contains only 10 capacity verification op_ids
- Rollback strategy
- Conclusion: `PASS`, `FAIL`, or `BLOCKED`

Do not create CP7 and do not call meta-qa. CP7 starts only after meta-po reviews CP6 and moves the Story to verification.

## Completion Criteria

- STORY-003 product files are implemented within the write scope.
- Required checks are run or explicitly blocked with reasons.
- CP6 file exists and includes dispatch evidence.
- The implementation establishes stable capacity domain contracts for downstream STORY-004, STORY-005, and STORY-006.
