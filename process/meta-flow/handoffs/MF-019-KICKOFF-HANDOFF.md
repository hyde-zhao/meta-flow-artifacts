---
handoff_id: "H-MF-019-KICKOFF"
cr_id: "MF-019"
title: "Permission / Security / Environment Governance"
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
  forbidden_question_scope: "runtime authorization, credential access, NAS access, QMT/MiniQMT/XtQuant connection, order write, provider/lake publish"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  context_ref: "process/context/MF-019.KICKOFF.context.json"
  read_profile: "minimal"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  allowed_reads:
    - "process/context/MF-019.KICKOFF.context.json"
    - "process/changes/summaries/MF-019.summary.json"
  read_if_needed:
    - "delivery/rules/AGENT-SKILL-CONTRACT.md"
    - "delivery/skills/context-manifest-builder/templates/AUTHZ-POLICY-TEMPLATE.json"
  do_not_read_by_default:
    - ".env"
    - "credentials"
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "full conversation transcript"
---

# MF-019 Kickoff Handoff

Start from `process/context/MF-019.KICKOFF.context.json`.

Objective: implement agent permission matrix, environment/toolchain profiles, secret checks, and redaction checks. This CR must not read credentials, connect runtime systems, access NAS, submit/cancel orders, or publish provider/lake/catalog data.

Expected outputs:

- `process/policies/AGENT-PERMISSION-MATRIX.yaml` template
- `process/policies/TOOLCHAIN-MANIFEST.yaml` template
- `process/policies/ENVIRONMENT-PROFILE.yaml` template
- `meta-flow permission check`
- `meta-flow security secret-check`
- `meta-flow security redaction-check`
- `meta-flow environment check`
- `tests/test_permission_environment_governance.py`

Non-goals:

- no global threat model requirement
- no full agent run manifest
- no real credential or runtime access

Verification:

- guardrail check
- full pytest
- install dry-run
