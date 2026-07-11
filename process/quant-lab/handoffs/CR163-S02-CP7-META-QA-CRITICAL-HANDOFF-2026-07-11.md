---
handoff_id: "CR163-S02-CP7-META-QA-CRITICAL-20260711"
workflow_id: "CR-163"
change_id: "CR-163"
story_id: "CR163-S02-recorder-seal-supersession"
stage: "story-execution"
checkpoint: "CP7"
status: "completed"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "independent-storage-integrity-recovery-verification"
  tool_name: "spawn_agent"
  agent_id: "/root/qa_critical_s02"
  thread_id: "/root/qa_critical_s02"
  spawned_at: "2026-07-11T14:37:00+08:00"
  completed_at: "2026-07-11T15:00:00+08:00"
question_permission: {can_ask_user: false, mode: "relay-via-host-orchestrator", structured_choice_allowed: false, broker_agent: "host-orchestrator"}
context_policy: {capsule_first: true, capsule_path: "process/context/stories/STORY-CR163-S02.CP7.work-packet.json", read_profile: "minimal"}
---

# S02 independent CP7 verification

Verify only; use adversarial crash/tamper/prefix/chain/path/concurrency probes. Do not fix source/tests.
