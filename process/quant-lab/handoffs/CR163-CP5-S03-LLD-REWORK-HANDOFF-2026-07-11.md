---
handoff_id: "CR163-CP5-S03-LLD-REWORK-20260711"
workflow_id: "CR-163"
change_id: "CR-163"
story_id: "CR163-S03-two-producer-chain-instrumentation"
stage: "story-planning"
checkpoint: "CP5"
status: "completed"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "default"
  dispatch_trigger: "user-requested-cp5-public-api-hardening"
  tool_name: "followup_task"
  agent_id: "/root/dev_zhu_integration"
  thread_id: "/root/dev_zhu_integration"
  resumed_at: "2026-07-11T12:20:00+08:00"
  completed_at: "2026-07-11T12:25:00+08:00"
question_permission: {can_ask_user: false, mode: "none", structured_choice_allowed: false, broker_agent: "host-orchestrator"}
context_policy: {capsule_first: true, capsule_path: "process/context/CP5-CR163-TRIAL-LINEAGE-INSTRUMENTATION-LLD-CONTEXT.yaml", read_profile: "minimal"}
---

# S03 LLD rework

Modify only the S03 LLD and its rework return summary. Freeze programmatic `ProducerLineageConfig | None`; CLI `--lineage-spec <local-json-path>` plus `--lineage-root <explicit-local-output-root>`; both absent means typed_unavailable; both present parse strict ExperimentFamilySpec before first trial; partial/invalid/path/schema config fails closed and cannot produce present; prohibit env/default-directory/history/manual-count inference; both chains share one parser/error contract. Do not implement code.
