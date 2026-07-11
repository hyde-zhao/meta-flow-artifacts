---
handoff_id: "CR163-CP4-META-SE-CRITICAL-20260711T112000+0800"
from_agent: "host-orchestrator"
to_agent: "meta-se"
workflow_id: "CR-163"
change_id: "CR-163"
phase: "story-planning"
checkpoint: "CP4"
status: "completed"
created_at: "2026-07-11T11:20:00+08:00"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-se"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "architecture-freeze-public-contract-and-cross-module-story-planning"
  agent_path: "agents/meta-se-critical.md"
  tool_name: "followup_task"
  agent_id: "se_chu_critical"
  agent_name: "Hilbert"
  thread_id: "se_chu_critical"
  spawned_at: ""
  resumed_at: "2026-07-11T11:20:00+08:00"
  completed_at: "2026-07-11T11:36:48+08:00"
  evidence: "followup_task"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Write non-blocking planning questions into the return summary; blocking scope or authorization questions must be relayed."
  forbidden_question_scope: "CP5 formal gate, runtime authorization, credentials, security-boundary expansion, publish, live or trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP4-CR163-TRIAL-LINEAGE-INSTRUMENTATION-STORY-PLANNING-CONTEXT.yaml"
  context_ref: "process/context/CP4-CR163-TRIAL-LINEAGE-INSTRUMENTATION-STORY-PLANNING-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 18
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads: "Use capsule context_policy.allowed_reads."
  must_read: "Use capsule context_policy.must_read."
  read_if_needed: "Use capsule context_policy.read_if_needed."
  do_not_read_by_default: "Use capsule context_policy.do_not_read_by_default."
---

# CR163 CP4 Meta-SE Critical Handoff

## Objective

Generate the formal CP4 Feature/Story planning package from the user-approved CP3 architecture, validate it, and return control to the Host Orchestrator. Do not implement source or tests.

## Required outputs

- `docs/design/FEATURE-DESIGN-MATRIX.md` or a CR163-scoped path consistent with current repository coexistence rules.
- Required `docs/features/<feature>/DESIGN.md`, `TEST-PLAN.md`, and `TASKS.md` artifacts.
- Five `process/stories/STORY-CR163-S0*-*.md` cards.
- `process/DEVELOPMENT-PLAN.yaml` updated as the machine truth source without damaging unrelated active plans.
- `process/checks/CP4-CR163-TRIAL-LINEAGE-INSTRUMENTATION-STORY-DAG-PARALLEL-SAFETY.result.json` and summary.
- `process/handoffs/CR163-CP4-META-SE-RETURN-SUMMARY.md`.

## Exit conditions

- Five Stories exactly cover S01-S05 and S03 covers both deduplicated chains plus CPI-CR163-001..004.
- Every Story has feature design refs, lld policy, quantitative acceptance, typed dependencies and file ownership.
- DAG is acyclic, file conflicts/merge order are explicit, and the all-Story CP5 design batch is computable.
- CP4 result is PASS with no unwaived blocker; otherwise return BLOCKED with exact route.
- No source/test/runtime/data/credential/external/release mutation.
