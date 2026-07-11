---
handoff_id: "CR163-CP8-META-QA-CRITICAL-RISK-RESPONSE-20260711"
workflow_id: "CR-163"
change_id: "CR-163"
stage: "documentation"
checkpoint: "CP8"
status: "completed"
dispatch: {required: true, semantic: "cp8-remediation", mode: "subagent", platform: "codex", canonical_role: "meta-qa", codex_agent_name: "meta-qa-critical", reasoning_profile: "critical", dispatch_trigger: "user-required-risk-response-triggers-before-cp8-approval", tool_name: "followup_task", agent_id: "/root/qa_critical_s04", thread_id: "/root/qa_critical_s04", spawned_at: "2026-07-11T08:30:00+00:00", completed_at: "2026-07-11T08:38:00+00:00"}
question_permission: {can_ask_user: false, mode: "relay-via-host-orchestrator", broker_agent: "host-orchestrator"}
context_policy: {capsule_first: true, capsule_path: "process/context/CP8-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml", read_profile: "compact"}
---
# CR163 CP8 risk-response remediation

Separate accepted risks, design boundaries and process controls; add executable follow-up triggers before CP8 closure.
