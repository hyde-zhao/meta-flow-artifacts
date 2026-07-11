---
handoff_id: "CR163-S05-CP7-META-QA-CRITICAL-20260711"
workflow_id: "CR-163"
change_id: "CR-163"
story_id: "CR163-S05-integrity-recovery-permission-regression"
stage: "story-execution"
checkpoint: "CP7"
status: "running"
dispatch: {required: true, semantic: "stage-dispatch", mode: "subagent", platform: "codex", canonical_role: "meta-qa", codex_agent_name: "meta-qa-critical", reasoning_profile: "critical", dispatch_trigger: "final-cross-story-integrity-and-permission-verification", tool_name: "spawn_agent", agent_id: "/root/qa_critical_s05", thread_id: "/root/qa_critical_s05", spawned_at: "2026-07-11T09:02:00+00:00", completed_at: ""}
question_permission: {can_ask_user: false, mode: "relay-via-host-orchestrator", broker_agent: "host-orchestrator"}
context_policy: {capsule_first: true, capsule_path: "process/context/stories/STORY-CR163-S05.CP7.work-packet.json", read_profile: "minimal"}
---
# S05 final independent CP7
Verify final cross-story evidence only; no fixes.
