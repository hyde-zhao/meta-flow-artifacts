---
handoff_id: "CR163-CP5-S02-LLD-REWORK-20260711"
workflow_id: "CR-163"
change_id: "CR-163"
story_id: "CR163-S02-recorder-seal-supersession"
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
  dispatch_trigger: "user-requested-cp5-lld-hardening"
  tool_name: "followup_task"
  agent_id: "/root/dev_yang_core"
  thread_id: "/root/dev_yang_core"
  resumed_at: "2026-07-11T12:20:00+08:00"
  completed_at: "2026-07-11T12:25:00+08:00"
question_permission: {can_ask_user: false, mode: "none", structured_choice_allowed: false, broker_agent: "host-orchestrator"}
context_policy: {capsule_first: true, capsule_path: "process/context/CP5-CR163-TRIAL-LINEAGE-INSTRUMENTATION-LLD-CONTEXT.yaml", read_profile: "minimal"}
---

# S02 LLD rework

Modify only the S02 LLD and its rework return summary. Preserve the existing fail-closed/no-resume design. Add: explicit long-running-search product impact; no in-place resume; new family required; 10k synthetic characterization for open/rebuild/seal/peak-memory/manifest-size; characterization is not a capacity promise; segmented ledger/incremental seal/sharded manifest require a follow-up storage CR. Do not implement code.
