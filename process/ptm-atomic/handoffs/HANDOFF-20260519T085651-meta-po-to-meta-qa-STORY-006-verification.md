---
handoff_id: "HANDOFF-20260519T085651-meta-po-to-meta-qa-STORY-006-verification"
from_agent: "meta-po"
to_agent: "meta-qa"
workflow_id: "atomic-ops-ngfw-install-config-chain"
change_id: "CR-003"
batch_id: "CP7-STORY-006"
story_id: "STORY-006"
wave_id: "W3"
task_type: "verification"
status: "completed"
created_at: "2026-05-19T08:56:51+0800"
reuse_decision:
  exact_reusable_agent_found: false
  previous_meta_qa_agent: "qa-yan"
  previous_meta_qa_thread_id: "019e3a91-a7c7-7920-814d-74c955d0ad70"
  decision: "spawn-new-meta-qa"
  rationale: "state-router exact reuse key is role+workflow_id+change_id+story_id+wave_id; existing qa-yan CP7 work was STORY-001..005/W1-W2, while this handoff is STORY-006/W3."
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-qa"
  agent_path: ".agents/agents/meta-qa.md"
  tool_name: "spawn_agent"
  agent_id: "019e3dc0-2f13-7023-8b53-5d25cbd95a05"
  agent_name: "qa-kong"
  thread_id: "019e3dc0-2f13-7023-8b53-5d25cbd95a05"
  spawned_at: "2026-05-19T09:00:52+0800"
  resumed_at: ""
  completed_at: "2026-05-19T09:06:45+0800"
  evidence: "host-spawned-new-meta-qa-for-STORY-006-CP7"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
---

# Handoff: STORY-006 CP7 Verification

## Dispatch Status

This file is a handoff request, not execution evidence. Host must use platform subagent dispatch and then write back dispatch evidence.

Recommended dispatch:

- `spawn_agent` a new `meta-qa` for `STORY-006` CP7.
- Do not resume `qa-yan` by default: `qa-yan` completed `STORY-001..005` CP7 under a different exact reuse key (`story_id=STORY-001..005`, `wave_id=W1-W2`).
- If the platform cannot spawn a new QA agent and explicitly chooses to resume `qa-yan`, record the actual `resume_agent` / `send_input` evidence in this handoff and explain the override in `dispatch.evidence`.

After real dispatch, update this frontmatter:

- `status`
- `dispatch.mode=subagent`
- `dispatch.tool_name=spawn_agent | resume_agent | send_input | resume_agent+send_input`
- `dispatch.agent_id`
- `dispatch.agent_name`
- `dispatch.thread_id`
- `dispatch.spawned_at` or `dispatch.resumed_at`
- `dispatch.completed_at` when finished
- `dispatch.evidence`

## Required Inputs

Read before verification:

- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/DEVELOPMENT-PLAN.yaml`
- `process/VALIDATION-ENV.yaml`
- `process/TEST-STRATEGY.md`
- `process/VERIFICATION-REPORT.md`
- `process/stories/STORY-006-update-user-facing-docs-and-release-guidance.md`
- `process/stories/STORY-006-update-user-facing-docs-and-release-guidance-LLD.md`
- `process/checks/CP5-STORY-006-update-user-facing-docs-and-release-guidance-LLD-IMPLEMENTABILITY.md`
- `process/checks/CP6-STORY-006-update-user-facing-docs-and-release-guidance-CODING-DONE.md`
- `process/checks/CP7-STORY-001-freeze-schema-v11-contract-and-field-docs-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-002-model-ngfw-install-init-login-guard-atoms-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-004-model-multi-device-batch-configuration-contract-VERIFICATION-DONE.md`
- `process/checks/CP7-STORY-005-add-read-only-security-gate-and-validation-checks-VERIFICATION-DONE.md`
- `README.md`
- `docs/USER-MANUAL.md`
- `docs/engineer-handbook.md`
- `docs/test-case-template.yaml`
- `CHANGELOG.md`
- Referenced current repository facts under `atoms/`, `packages/`, `schemas/`, `scripts/`, and `src/atomic_ops/`.

## Task

Verify `STORY-006` only. Do not implement or edit product docs.

Create:

- `process/checks/CP7-STORY-006-update-user-facing-docs-and-release-guidance-VERIFICATION-DONE.md`

Update if needed:

- `process/VERIFICATION-REPORT.md` to include STORY-006 verification result.

Do not update:

- `process/STATE.md`
- `process/STORY-STATUS.md`
- `process/DEVELOPMENT-PLAN.yaml`
- Story cards
- Product files
- CP8 or documentation-phase files

## Verification Scope

Verify that the STORY-006 documentation implementation satisfies the accepted Story and LLD:

- Five target docs are present and updated: `README.md`, `docs/USER-MANUAL.md`, `docs/engineer-handbook.md`, `docs/test-case-template.yaml`, `CHANGELOG.md`.
- README positive delivery surface stays native to the repository and does not present `delivery/` as a user-facing delivery directory.
- CLI examples use only shipped read-only commands: `sync`, `list`, `show`, `packages`, `validate`, help/version, install, and repository checks.
- Python/script examples use `uv run`, `uv run --python 3.11 python ...`, or `uv tool`; default bare `pip install` count is 0.
- Docs state the CLI does not connect to devices, execute atoms, push configuration, save credentials, read `.input` data, or perform automatic rollback.
- Package and `op_id` examples resolve from current repository files.
- Sensitive values are absent; allowed exception is the documented password-policy value `Ngfw@123` if present as policy text only.
- Known `atomic-ops packages` synced-cache limitation is treated consistently with prior CP7 results: cache limitation may be non-blocking only if working-tree package/op_id reference probes show zero missing refs.

## Required Checks

Run or justify equivalent read-only checks:

- `uv run atomic-ops --help`
- `uv run atomic-ops packages`
- `uv run --python 3.11 python scripts/validate_schema.py atoms`
- `uv run --python 3.11 python scripts/layout_check.py`
- `uv run --python 3.11 python scripts/security_gate_check.py`
- Working-tree package/op_id reference probe for documented examples.
- Scan target docs for unsupported `atomic-ops` verbs, positive `delivery/` instructions, bare `pip install` default usage, real IP/token/cookie/header/FTP credential shapes, and misleading device execution language.

## CP7 Output Requirements

`process/checks/CP7-STORY-006-update-user-facing-docs-and-release-guidance-VERIFICATION-DONE.md` must use checkpoint-manager structure:

- Entry Criteria
- Checklist
- Exit Criteria
- Deliverables
- Agent Dispatch Evidence
- Commands run and results
- Verification evidence for each STORY-006 acceptance criterion
- Non-blocking caveats, if any
- Conclusion

Conclusion may be `PASS`, `BLOCKED`, `FAIL`, or `WAIVED` according to evidence. Mark `PASS` only if all blocking verification criteria pass and dispatch evidence is complete. Do not start CP8 and do not enter documentation phase.
