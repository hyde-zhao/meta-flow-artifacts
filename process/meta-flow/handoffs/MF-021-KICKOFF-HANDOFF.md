---
handoff_id: "H-MF-021-KICKOFF"
cr_id: "MF-021"
title: "Release / Compatibility / Human Gate UX Governance"
status: "handoff-ready"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "meta-dev"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "default"
  dispatch_trigger: "phase-default"
  tool_name: ""
  agent_id: ""
  thread_id: ""
  evidence: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  forbidden_question_scope: "runtime authorization, credentials, security boundary, publish, live trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  context_ref: "process/context/MF-021.KICKOFF.context.json"
  read_profile: "minimal"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  allowed_reads:
    - "process/context/MF-021.KICKOFF.context.json"
    - "process/changes/summaries/MF-021.summary.json"
  read_if_needed:
    - "delivery/skills/release-readiness/SKILL.md"
    - "meta_flow/design/product_governance.py"
    - "meta_flow/checks/human_gate.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "full conversation transcript"
---

# MF-021 Kickoff Handoff

Start from `process/context/MF-021.KICKOFF.context.json`.

Objective: implement release policy, compatibility policy, package/API/CLI/legacy alias checks, partial approval, and blocking/non-blocking human gate grouping. Reuse existing human gate and checkpoint manager contracts; do not create a separate large decision UX state machine.

Expected outputs:

- `process/policies/RELEASE-POLICY.yaml` template
- `process/policies/COMPATIBILITY-POLICY.yaml` template
- `meta-flow release compatibility-check`
- `meta-flow human-gate decision-ux-check`
- `tests/test_release_compatibility_human_gate.py`

Non-goals:

- no default deprecation ledger
- no full release workflow for docs-lite/process-lite
- no runtime authorization by implication

Verification:

- guardrail check
- full pytest
- install dry-run
