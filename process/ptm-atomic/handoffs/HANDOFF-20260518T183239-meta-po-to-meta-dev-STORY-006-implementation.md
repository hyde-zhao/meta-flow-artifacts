---
handoff_id: "HANDOFF-20260518T183239-meta-po-to-meta-dev-STORY-006-implementation"
from_agent: "meta-po"
to_agent: "meta-dev"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
batch_id: "W3-dev-after-CP7"
story_id: "STORY-006"
wave_id: "W3"
task_type: "implementation"
status: "completed"
created_at: "2026-05-18T18:32:39+0800"
preferred_existing_agent:
  agent_name: "dev-xu"
  agent_id: "019e39fc-f3e8-7381-beae-85bef34273ab"
  thread_id: "019e39fc-f3e8-7381-beae-85bef34273ab"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  agent_path: ".agents/agents/meta-dev.md"
  tool_name: "resume_agent+send_input"
  agent_id: "019e39fc-f3e8-7381-beae-85bef34273ab"
  agent_name: "dev-xu"
  thread_id: "019e39fc-f3e8-7381-beae-85bef34273ab"
  spawned_at: ""
  resumed_at: "2026-05-19T08:45:21+0800"
  completed_at: "2026-05-19T08:55:24+0800"
  evidence: "host-resumed-existing-meta-dev-for-STORY-006-implementation"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: STORY-006 Implementation

## Dispatch Status

This file is a handoff request, not execution evidence. Host must resume the existing `dev-xu` meta-dev thread and then write back dispatch evidence.

Required dispatch:

- `resume_agent` / `send_input` to existing `dev-xu`
- `agent_id`: `019e39fc-f3e8-7381-beae-85bef34273ab`
- `thread_id`: `019e39fc-f3e8-7381-beae-85bef34273ab`
- Do not spawn another meta-dev unless `dev-xu` is unrecoverable.

After real dispatch, update this frontmatter:

- `status`
- `dispatch.mode=subagent`
- `dispatch.tool_name=resume_agent | send_input | resume_agent+send_input`
- `dispatch.agent_id`
- `dispatch.agent_name`
- `dispatch.thread_id`
- `dispatch.resumed_at`
- `dispatch.completed_at` when finished
- `dispatch.evidence`

## Required Inputs

`dev-xu` must read these before editing:

- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md`
- `process/ARCHITECTURE-DECISION.md`
- `process/PLATFORM-INSTALL-SPEC.md`
- `process/VALIDATION-ENV.yaml`
- `process/TEST-STRATEGY.md`
- `process/VERIFICATION-REPORT.md`
- `process/stories/STORY-006-update-user-facing-docs-and-release-guidance.md`
- `process/stories/STORY-006-update-user-facing-docs-and-release-guidance-LLD.md`
- `process/checks/CP5-STORY-006-update-user-facing-docs-and-release-guidance-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md`
- `process/checks/CP6-STORY-002-model-ngfw-install-init-login-guard-atoms-CODING-DONE.md`
- `process/checks/CP6-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-CODING-DONE.md`
- `process/checks/CP6-STORY-004-model-multi-device-batch-configuration-contract-CODING-DONE.md`
- `process/checks/CP6-STORY-005-add-read-only-security-gate-and-validation-checks-CODING-DONE.md`
- `process/checks/CP7-STORY-001-freeze-schema-v11-contract-and-field-docs-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-002-model-ngfw-install-init-login-guard-atoms-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-004-model-multi-device-batch-configuration-contract-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-005-add-read-only-security-gate-and-validation-checks-VERIFICATION-DONE.md`
- Current product files in allowed write scope and referenced upstream files under `atoms/`, `packages/`, `schemas/`, `docs/`, `scripts/`, `src/atomic_ops/`.

## Task

Implement `STORY-006` only: update user-facing docs and release guidance after STORY-001..005 CP6/CP7 PASS.

Use actual repository facts, not planned LLD assumptions. The docs must describe only shipped and verified behavior. The CLI remains offline/read-only: no device connection, atom execution, configuration push, credential storage, or automatic rollback.

## Unique Write Scope

`dev-xu` may write only:

- `README.md`
- `docs/USER-MANUAL.md`
- `docs/engineer-handbook.md`
- `docs/test-case-template.yaml`
- `CHANGELOG.md`
- `process/checks/CP6-STORY-006-update-user-facing-docs-and-release-guidance-CODING-DONE.md`

## Forbidden Write Scope

Do not modify:

- `.input/`
- `delivery/`
- `atoms/`
- `packages/`
- `schemas/`
- `src/`
- `scripts/`
- `pyproject.toml`
- `uv.lock`
- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/DEVELOPMENT-PLAN.yaml`
- other `process/stories/*.md`
- existing CP5/CP6/CP7 files for STORY-001..005
- CP7 or CP8 files
- `checkpoints/`

If an allowed doc needs a product fact not available from the repository, leave the claim out or record it as not shipped; do not invent behavior.

## Implementation Requirements

- Update all 5 target documents listed in the write scope.
- README positive delivery surface must stay native to this repository: `atoms/`, `schemas/`, `packages/`, `docs/`, `src/atomic_ops/`, `scripts/`, `pyproject.toml`, `uv.lock`; do not present `delivery/` as a user-facing delivery directory.
- Commands must use `uv run` or `uv tool`; do not use bare `pip install` as the default path.
- CLI command examples must be limited to shipped read-only surfaces: `sync`, `list`, `show`, `packages`, `validate`, help/version, and repository checks.
- Package and `op_id` examples must exist in the current repository and align with STORY-001..005 CP7 facts.
- Do not include real IP addresses, token/cookie examples, Authorization headers, FTP credentials, raw default passwords, live device access, or `.input/` data. The only explicitly allowed password-policy value is `Ngfw@123`.
- Preserve the boundary that validation failure produces diagnostics and manual remediation guidance; do not describe automatic device rollback as shipped behavior.

## Required Verification

Run the strongest applicable local read-only checks before writing CP6. At minimum:

- `uv run atomic-ops --help`
- `uv run atomic-ops packages`
- `uv run --python 3.11 python scripts/validate_schema.py atoms`
- `uv run --python 3.11 python scripts/layout_check.py`
- `uv run --python 3.11 python scripts/security_gate_check.py`

Also perform document-specific checks:

- Verify package ids referenced in docs exist under `packages/*.yaml`.
- Verify `op_id` examples referenced in docs exist under `atoms/*/*.yaml`.
- Verify docs do not introduce positive `delivery/` delivery-surface instructions.
- Verify docs do not introduce real device action commands or unsupported CLI verbs.
- Verify docs do not introduce sensitive values.

If `atomic-ops validate --package <package>` is affected by the known synced-cache limitation, record it as non-blocking only when a working-tree package/op_id reference probe shows 0 missing references, consistent with CP7 QA handling.

## CP6 Output Requirements

Create `process/checks/CP6-STORY-006-update-user-facing-docs-and-release-guidance-CODING-DONE.md` with checkpoint-manager structure:

- Entry Criteria
- Checklist
- Exit Criteria
- Deliverables
- Agent Dispatch Evidence
- Changed files mapped to STORY-006 TASK IDs
- Commands run and results
- Document command/reference/safety evidence
- Rollback plan
- Conclusion

The CP6 conclusion must be factual. Mark `PASS` only if all required docs are updated and checks pass or non-blocking caveats are explicitly justified. Do not create CP7, do not start meta-qa, and do not enter documentation phase.

## Acceptance Criteria

- 5 target docs are updated: missing count 0.
- README command surface includes only shipped read-only CLI commands plus help/version/install/check commands.
- Python/script examples use `uv run` or `uv tool`; default bare `pip install` count 0.
- README positive delivery surface excludes `delivery/`.
- Sensitive findings in docs are 0, except allowed `Ngfw@123` policy value.
- Referenced package ids and `op_id` examples resolve from current repository files.
- Docs explicitly state the CLI does not connect to devices, execute atoms, push configuration, or save credentials.
