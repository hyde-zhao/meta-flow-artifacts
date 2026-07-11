---
handoff_id: "CR163-CP2-META-PM-HANDOFF-2026-07-11"
workflow_id: "CR-163"
change_id: "CR-163"
cr_id: "CR-163"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
phase: "requirement-clarification"
checkpoint: "CP1/CP2"
semantic: "delegated-user-interaction"
status: "returned-with-questions"
created_at: "2026-07-11T10:22:00+08:00"
return_summary_path: "process/handoffs/CR163-CP2-META-PM-RETURN-SUMMARY.md"
reuse_key: "meta-pm|CR-163|CR-163||"
close_after: "CP2 decision brief inputs returned to host-orchestrator"
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
  agent_path: ".agents/agents/meta-pm.md"
  tool_name: "spawn_agent"
  agent_id: "/root/pm_wu"
  agent_name: "pm-wu"
  thread_id: "/root/pm_wu"
  spawned_at: "2026-07-11T10:23:00+08:00"
  resumed_at: ""
  completed_at: "2026-07-11T11:50:00+08:00"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Scenario Gray Areas, entrypoint inventory ambiguity, count semantics, scope and quantitative acceptance criteria; return a compact question batch to host-orchestrator."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 formal gates, runtime authorization, credentials, real data/NAS/provider access, publish, live/trading/broker authorization."
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR163.context.json"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 18
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Product baseline documents may be expanded only for the authorized incremental CR163 revision; complete historical CRs remain deny-default."
  allowed_reads:
    - "process/context/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml"
    - "process/context/CP2-CR163.context.json"
    - "process/changes/summaries/CR-163.summary.json"
    - "process/checks/CP0-CR163.route-plan.json"
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
    - "engine/mature_multifactor_research.py"
    - "engine/research_manifest.py"
    - "engine/backtest_production_contracts.py"
    - "engine/strategy_admission_statistical_gate.py"
    - "engine/cross_strategy_reliability_gates.py"
  must_read:
    - "process/context/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml"
    - "process/context/CP2-CR163.context.json"
    - "process/changes/summaries/CR-163.summary.json"
    - "process/checks/CP0-CR163.route-plan.json"
  read_if_needed:
    - "docs/product/* only for incremental revisions"
    - "engine/**/*.py only after rg identifies a candidate-producing or lineage-consumer path"
    - "tests/**/*.py only for fixture/static adapter and regression inventory"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "process/archive/**"
    - "process/discussions/** except the new CR163 discussion log the agent creates"
---

# CR163 CP2 Meta-PM Handoff

## Objective

Turn the approved FU-CR161-001 scope definition into an incremental, auditable CR163 product baseline and return CP1/CP2 decision inputs. Establish facts and traceability; do not design implementation details or write source code.

## Required Work

1. Read the capsule first and obey its authorization/read-budget policy.
2. Inspect repository facts to produce a named candidate-producing entrypoint inventory. Start from the five known engine files, use `rg` to find additional construction/consumer paths, and explicitly classify included/excluded paths. Do not scan or rewrite every historical script.
3. Incrementally update the eight product documents listed in the capsule. Preserve old baselines and append revision records; do not replace documents wholesale.
4. Cover positive, negative, boundary, permission, failure-recovery, retry/duplicate/seed, seal/supersede/tamper, missing-instrumentation and CR155 negative-regression scenarios.
5. Freeze CP2-level semantics: lifecycle, count definitions, availability states, effective-trial boundary, quantifiable acceptance criteria and non-authorization scope. Leave final type names/artifact architecture to CP3.
6. Produce the required CP1 result, CP2 discussion log/checkpoint, and return summary. If a user choice remains necessary, write a compact SGQ question batch with recommendation/options/impact; do not open CP2 yourself.

## Required Outputs

- Incrementally revised `docs/product/USE-CASES.md`
- Incrementally revised `docs/product/REQUIREMENTS.md`
- Incrementally revised `docs/product/SCENARIOS.yaml`
- Incrementally revised `docs/product/TEST-MATRIX.md`
- Incrementally revised `docs/product/STORY-MAP.md`
- Incrementally revised `docs/product/MVP-SCOPE.md`
- Incrementally revised `docs/product/RELEASE-SLICES.md`
- Incrementally revised `docs/product/BACKLOG.md`
- `process/discussions/CP2-CR163-SCENARIO-DISCUSSION-LOG.md`
- `process/checks/CP2-CR163-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP1-CR163-TRIAL-LINEAGE-INSTRUMENTATION-USE-CASE-COMPLETENESS.result.json`
- `process/handoffs/CR163-CP2-META-PM-RETURN-SUMMARY.md`

## Exit Criteria

- Frozen inventory and P0 coverage definition are explicit and repository-evidenced.
- User-provided count/availability/effective-trial/CR155 boundaries are preserved without statistical overclaim.
- All product-doc changes are incremental and revision-traceable.
- All open questions are classified, and host-orchestrator has enough evidence to run CP1 checks and prepare the CP2 Decision Brief.
- No Story decomposition, LLD, implementation, real data/runtime operation or formal human gate is performed.
