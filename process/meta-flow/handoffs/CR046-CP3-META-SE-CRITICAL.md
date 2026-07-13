---
handoff_id: "HO-CR046-CP3-META-SE-CRITICAL"
from_agent: "host-orchestrator"
to_agent: "meta-se"
workflow_id: "meta-flow-self-dev"
change_id: "CR-046"
stage: "solution-design"
status: "completed"
created_at: "2026-07-12T01:43:13Z"
semantic: "delegated-user-interaction"
return_summary_path: "process/handoffs/CR046-CP3-META-SE-CRITICAL-RETURN-SUMMARY.md"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-se"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "architecture-freeze-public-evidence-contract"
  agent_path: "delivery/agents/meta-se.md"
  tool_name: "spawn_agent"
  agent_id: "/root/se_critical_cr046"
  agent_name: "se_critical_cr046"
  thread_id: "/root/se_critical_cr046"
  spawned_at: "2026-07-12T01:46:47Z"
  resumed_at: ""
  completed_at: "2026-07-12T01:53:44Z"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Architecture gray areas, trade-offs and CP3 decision candidates"
  forbidden_question_scope: "CP3 formal approval, runtime, credentials, publication, quant-lab business-code changes"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP3-CR046-DESIGN-CONTEXT.yaml"
  context_ref: "process/context/CP3-CR046-DESIGN-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 16
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "architecture-major-contract-freeze"
  allowed_reads: ["process/context/CP3-CR046-DESIGN-CONTEXT.yaml", "process/state/STATE.current.json", "process/changes/summaries/CR-046.summary.json", "process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json", "process/checkpoints/CP2-CR046-REQUIREMENTS-BASELINE-R2.md", "process/reviews/CR046-CP2-SCOPE-REVIEW-R1.md", "docs/product/*", "docs/design/*", "meta_flow/state/ledger_compaction.py", "meta_flow/checks/cp_result.py", "meta_flow/state/current.py"]
  must_read: ["process/context/CP3-CR046-DESIGN-CONTEXT.yaml", "process/state/STATE.current.json", "process/changes/summaries/CR-046.summary.json", "process/checks/CP2-CR046-REQUIREMENTS-BASELINE-R2.result.json", "process/checkpoints/CP2-CR046-REQUIREMENTS-BASELINE-R2.md"]
  read_if_needed: ["/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/product/*", "/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/design/*", "process/changes/CR-046.md", "meta_flow/**", "tests/**"]
  do_not_read_by_default: ["process/STATE.md", "process/DEVELOPMENT-PLAN.yaml", "process/archive/**", "process/discussions/**", "process/stories/**", "quant-lab source code", "complete conversation transcript"]
---

# CR-046 CP3 Meta-SE-Critical Handoff

Design the architecture-major evidence-integrity contract and return control with CP3 automatic evidence. The CP3 Decision Brief must include a dedicated architecture decision for `routing_ref` local-directory compatibility: primary recommendation, executable alternatives, valid minimum metadata, dangling-ref fail behavior, workspace/state checker shared truth, migration/rollback and switch conditions. It must not be deferred to CP8.

Do not launch or approve CP3. Do not implement code, create LLDs, commit/push, access credentials/runtime, modify quant-lab business code or execute the CR-163 pilot.
