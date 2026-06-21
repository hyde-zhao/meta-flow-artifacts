---
handoff_id: "H-MF-018-KICKOFF"
cr_id: "MF-018"
title: "Quality / Eval / Minimal Metrics Governance"
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
  forbidden_question_scope: "CP2/CP3/CP5/CP8 formal gates, runtime authorization, credentials, security boundary, publish, live trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  context_ref: "process/context/MF-018.KICKOFF.context.json"
  read_profile: "minimal"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  allowed_reads:
    - "process/context/MF-018.KICKOFF.context.json"
    - "process/changes/summaries/MF-018.summary.json"
  read_if_needed:
    - "README.md"
    - "delivery/README.md"
    - "delivery/rules/AGENT-SKILL-CONTRACT.md"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "full conversation transcript"
---

# MF-018 Kickoff Handoff

Start from `process/context/MF-018.KICKOFF.context.json`.

Objective: implement lightweight quality model, eval matrix, and minimal workflow metrics. Keep metrics derived from existing CP results, event ledgers, read expansion ledgers, CR ledgers, and story ledgers. Do not create a manually maintained workflow metrics truth source.

Expected outputs:

- `process/policies/QUALITY-MODEL.yaml` template
- `process/policies/EVAL-MATRIX.yaml` template
- `meta-flow quality model-check`
- `meta-flow quality eval-check`
- `meta-flow doctor quality`
- `meta-flow doctor workflow`
- `tests/test_quality_governance.py`

Non-goals:

- no dashboard
- no agent performance ranking
- no portfolio reporting

Verification:

- `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_delivery_guardrails.py`
- `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --with pytest --python 3.11 pytest -q`
- `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow install --platform codex --scope project --component agent --project-dir . --dry-run`
