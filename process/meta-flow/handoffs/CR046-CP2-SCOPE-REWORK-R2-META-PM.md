---
handoff_id: "HO-CR046-CP2-SCOPE-REWORK-R2-META-PM"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
workflow_id: "meta-flow-self-dev"
change_id: "CR-046"
stage: "requirement-clarification-rework"
status: "completed"
created_at: "2026-07-12T01:19:18Z"
semantic: "stage-dispatch"
return_summary_path: "process/handoffs/CR046-CP2-SCOPE-REWORK-R2-META-PM-RETURN-SUMMARY.md"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "cp2-changes-requested-scope-normalization"
  agent_path: "delivery/agents/meta-pm.md"
  tool_name: "followup_task"
  agent_id: "/root/pm_cr046_r2"
  agent_name: "pm_cr046_r2"
  thread_id: "/root/pm_cr046_r2"
  spawned_at: "2026-07-12T01:20:45Z"
  resumed_at: ""
  completed_at: "2026-07-12T01:29:00Z"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Only a newly discovered material conflict in the accepted scope additions"
  forbidden_question_scope: "CP2 approval, runtime, credentials, publication, quant-lab business-code modification"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "cp2_scope_rework"
  allowed_reads: ["process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml", "process/reviews/CR046-CP2-SCOPE-REVIEW-R1.md", "process/changes/CR-046.md", "docs/product/*"]
  must_read: ["process/context/CP2-CR046-REQUIREMENT-CONTEXT.yaml", "process/reviews/CR046-CP2-SCOPE-REVIEW-R1.md", "process/changes/CR-046.md"]
  read_if_needed: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]
  do_not_read_by_default: ["process/STATE.md", "process/DEVELOPMENT-PLAN.yaml", "process/archive/**", "process/stories/**", "quant-lab source code"]
---

# CR-046 CP2 Scope Rework R2

Incrementally normalize the product baseline for the five accepted findings in `process/reviews/CR046-CP2-SCOPE-REVIEW-R1.md`. Preserve every existing UC/REQ/TC/ST ID and revision history. Add or extend requirements/scenarios/stories so the following are explicit required scope: compaction semantic preservation, reusable post-close correction lifecycle, machine-generated provenance-bearing audit report, CP1/CP2 null-provenance dogfooding, and session-observed/repository-unverifiable dispatch disclosure.

Rerun CP1 and CP2 as new R2 results; do not overwrite the R1 result files. Generate a refreshed CP2 capsule and return summary. Do not approve or launch CP2.
