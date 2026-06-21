---
handoff_id: "H-MF-020-KICKOFF"
cr_id: "MF-020"
title: "Concurrency / Recovery / Schema Migration Governance"
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
  forbidden_question_scope: "formal human gates, runtime authorization, credentials, trading, publish"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  context_ref: "process/context/MF-020.KICKOFF.context.json"
  read_profile: "minimal"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  allowed_reads:
    - "process/context/MF-020.KICKOFF.context.json"
    - "process/changes/summaries/MF-020.summary.json"
  read_if_needed:
    - "delivery/skills/change-impact-analysis/templates/CR-INDEX-TEMPLATE.yaml"
    - "meta_flow/checks/cr_tracking.py"
    - "meta_flow/workflow/cr_lifecycle.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "full conversation transcript"
---

# MF-020 Kickoff Handoff

Start from `process/context/MF-020.KICKOFF.context.json`.

Objective: implement dynamic concurrency checks, recovery policy, schema migration policy, and standardized CR relationship fields. Reuse existing Story packet, Story return, Feature Registry, Module Boundaries, CR Index, and CR Ledger facts. Do not create stale persistent file or contract lock truth sources, and do not create a separate CR relationship registry unless later evidence shows CR-INDEX / CR summary cannot carry the relationship model.

Expected outputs:

- `meta-flow concurrency check`
- `meta-flow recovery policy-check`
- `meta-flow schema policy-check`
- CR relationship template/schema fields
- CR relationship validator for dependency, reference, blocking, supersession, enablement, weak relation, and derivation semantics
- `tests/test_concurrency_recovery_schema.py`

Important design constraint:

CR relationships must support only this controlled vocabulary:

| Relationship | Meaning | Blocks start |
|---|---|---|
| `depends_on` | Hard dependency; prerequisite CR must be completed or explicitly satisfied before this CR starts. | yes |
| `recommended_after` | Recommended order; parallel execution is allowed with rationale. | no |
| `references` | Background, policy, upstream decision, or historical context reference. | no |
| `blocks` | Current CR blocks another CR, gate, capability, release, or follow-up item. | conditional |
| `blocked_by` | Current CR is blocked by a CR, gate, risk, missing authorization, or unresolved decision. | yes |
| `supersedes` | Current CR replaces an older CR or candidate item. | no, but old item should close or stop |
| `superseded_by` | Current CR has been replaced by another CR. | yes |
| `enables` | Current CR unlocks a later CR, capability, profile, or workflow path after completion. | no |
| `related_to` | Weak audit/search relation only. | no |
| `derived_from` | Current CR was derived from another CR, CP decision, follow-up candidate, spike candidate, issue, or user request. | no |

Conflict detection must continue to use:

- `conflict_keys`
- `impact_surface`
- `affected_docs`
- `affected_features`
- `affected_module_paths`
- `authz_policy_refs`
- `risk_refs`

Do not rely only on manually declared relationships such as `depends_on` or `related_to` for conflict detection.

Acceptance details:

- `depends_on` target missing or not satisfied must block CR start.
- `blocked_by` must block CR start until the blocker is closed, waived by policy, or converted to a non-blocking risk with explicit approval.
- `superseded_by` must prevent further execution of the superseded CR.
- `recommended_after` violations should warn unless an explicit parallelization rationale is present.
- `references`, `related_to`, and `derived_from` are audit/search metadata and must not become hidden execution locks.
- `blocks` must explain what is blocked and whether that target is a CR, gate, release, capability, or follow-up item.

Verification:

- guardrail check
- full pytest
- install dry-run
