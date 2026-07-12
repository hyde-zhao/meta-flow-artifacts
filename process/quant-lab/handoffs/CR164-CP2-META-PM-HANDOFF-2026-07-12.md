---
handoff_id: "CR164-CP2-META-PM-HANDOFF-2026-07-12"
workflow_id: "CR-164"
change_id: "CR-164"
cr_id: "CR-164"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
phase: "requirement-clarification"
checkpoint: "CP1/CP2"
semantic: "delegated-user-interaction"
status: "returned"
created_at: "2026-07-12T17:46:02+08:00"
return_summary_path: "process/handoffs/CR164-CP2-META-PM-RETURN-SUMMARY.md"
reuse_key: "meta-pm|CR-164|CR-164||"
close_after: "CP2 Decision Brief inputs returned to host-orchestrator"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "phase-default"
  agent_path: ".claude/agents/meta-pm.md"
  tool_name: "spawn_agent"
  agent_id: "/root/pm_wu"
  agent_name: "pm-wu"
  thread_id: "/root/pm_wu"
  spawned_at: "2026-07-12T17:49:00+08:00"
  resumed_at: "2026-07-12T18:05:00+08:00"
  completed_at: "2026-07-12T18:35:00+08:00"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Scenario Gray Areas, MVP statistical-method scope, input sufficiency thresholds, compatibility scope, evidence/availability semantics and quantitative acceptance criteria; return a compact SGQ batch to the host."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 formal gates, runtime authorization, credentials, production datasets/NAS/provider access, publish, live/trading/broker authorization."
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR164.context.json"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 18
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Only the eight explicitly affected product baselines may be expanded for incremental CR164 revision."
  allowed_reads:
    - "process/context/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-CONTEXT.yaml"
    - "process/context/CP2-CR164.context.json"
    - "process/changes/summaries/CR-164.summary.json"
    - "process/checks/CP0-CR164.route-plan.json"
    - "process/checks/CP0-CR-164-BOOTSTRAP.result.json"
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
    - "engine/strategy_admission_statistical_gate.py"
    - "engine/cross_strategy_reliability_gates.py"
    - "engine/anomaly_multiple_testing.py"
    - "engine/experiment_family_lineage.py"
    - "engine/strategy_admission_package.py"
  must_read:
    - "process/context/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-CONTEXT.yaml"
    - "process/context/CP2-CR164.context.json"
    - "process/changes/summaries/CR-164.summary.json"
    - "process/checks/CP0-CR164.route-plan.json"
    - "process/checks/CP0-CR-164-BOOTSTRAP.result.json"
  read_if_needed:
    - "docs/product/* only for incremental CR164 revisions"
    - "engine/**/*.py only after rg identifies a relevant calculator, contract or consumer"
    - "tests/**/*.py only for fixture/static behavior inventory"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "process/archive/**"
    - "process/discussions/** except the new CR164 discussion log"
---

# CR164 CP2 Meta-PM Handoff

## Objective

Turn the explicitly activated `FU-CR161-002` into an incremental, auditable CR164 product baseline for computable multiple-testing / PBO / DSR evidence. Establish user value, scenarios, method-scope choices, quantitative acceptance criteria, fail-closed behavior and traceability. Do not design implementation details or write source code.

## Required Work

1. Read the capsule first and obey its read/authorization budget.
2. Establish current repository facts for existing report contracts, evidence slots, CR163 lineage inputs and any already-implemented BH logic. Use `rg` before opening additional source files.
3. Incrementally update the eight product documents listed in the capsule. Preserve previous baselines and append revision records; do not replace documents wholesale.
4. Cover positive, negative, boundary, permission and failure-recovery scenarios, including incomplete lineage, low sample/fold/trial counts, NaN/Inf/degenerate inputs, method disagreement, hash/ref mismatch and CR155 negative regression.
5. Prepare CP2-level decisions for MVP method set, input sufficiency, effective-trial boundary, compatibility scope and quantitative AC. Keep final schema/module design for CP3.
6. Produce the required CP1 result, discussion log/checkpoint and return summary. If user input is needed, return a compact SGQ batch with recommendation, alternatives and impact; do not open CP2 yourself.

## Required Outputs

- Incrementally revised `docs/product/USE-CASES.md`
- Incrementally revised `docs/product/REQUIREMENTS.md`
- Incrementally revised `docs/product/SCENARIOS.yaml`
- Incrementally revised `docs/product/TEST-MATRIX.md`
- Incrementally revised `docs/product/STORY-MAP.md`
- Incrementally revised `docs/product/MVP-SCOPE.md`
- Incrementally revised `docs/product/RELEASE-SLICES.md`
- Incrementally revised `docs/product/BACKLOG.md`
- `process/discussions/CP2-CR164-SCENARIO-DISCUSSION-LOG.md`
- `process/checks/CP2-CR164-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP1-CR164-MULTIPLE-TESTING-PBO-DSR-USE-CASE-COMPLETENESS.result.json`
- `process/handoffs/CR164-CP2-META-PM-RETURN-SUMMARY.md`

## Exit Criteria

- P0 journey, method/input inventory, scope candidates, quantitative AC, exclusions and fail-closed semantics are explicit and repository-evidenced.
- All product-document changes are incremental and revision-traceable.
- All open questions are classified and the host has enough evidence to prepare CP2.
- No Story decomposition, LLD, implementation, real data/runtime operation or formal human gate is performed.
