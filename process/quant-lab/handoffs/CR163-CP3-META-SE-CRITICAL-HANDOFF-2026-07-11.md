---
handoff_id: "CR163-CP3-META-SE-CRITICAL-HANDOFF-2026-07-11"
workflow_id: "CR-163"
change_id: "CR-163"
cr_id: "CR-163"
from_agent: "host-orchestrator"
to_agent: "meta-se-critical"
phase: "solution-design"
checkpoint: "CP3"
semantic: "delegated-user-interaction"
status: "completed"
created_at: "2026-07-11T10:54:00+08:00"
return_summary_path: "process/handoffs/CR163-CP3-META-SE-RETURN-SUMMARY.md"
reuse_key: "meta-se|CR-163|CR-163||"
close_after: "CP3 design return integrated and CP3 human gate opened"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-se"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "architecture-freeze"
  agent_path: ".agents/agents/meta-se-critical.md"
  tool_name: "spawn_agent"
  agent_id: "/root/se_chu_critical"
  agent_name: "se-chu-critical"
  thread_id: "/root/se_chu_critical"
  spawned_at: "2026-07-11T10:55:00+08:00"
  resumed_at: ""
  completed_at: "2026-07-11T11:42:00+08:00"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Architecture Gray Areas, public contracts, module/storage/integration boundaries and CP3 decision alternatives; return DQ batch to host."
  forbidden_question_scope: "Formal CP3/CP5/CP8 gate, implementation authorization, credentials, real data/runtime/trading/broker/publish authorization."
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-DESIGN-CONTEXT.yaml"
  context_ref: "process/context/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-DESIGN-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 22
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Architecture-major design may expand only named approved product/design/source contracts."
  allowed_reads: ["CP3 capsule allowed_reads only"]
  must_read: ["CP3 capsule", "CR163 summary", "approved CP2 checkpoint", "confirmed SGQ checkpoint"]
  read_if_needed: ["named product baselines", "CR161 HLD/ADR baseline", "named engine/scripts producer and consumer contracts"]
  do_not_read_by_default: ["process/STATE.md", "process/DEVELOPMENT-PLAN.yaml", "process/changes/CR-*.md", "process/stories/**", "process/archive/**", "docs/quality/**", "complete transcript"]
---

# CR163 CP3 Meta-SE-Critical Handoff

## Objective

Produce a reviewable CP3 blueprint/HLD/ADR for the approved CR163 trial-lineage instrumentation scope. Freeze the public contracts, lifecycle/state/count semantics, append-only storage and deterministic seal/supersession protocol, two-producer-chain integration, admission-consumer boundary and five-Story CP4 inputs.

## Hard Requirements

- Read the CP3 capsule first and obey its allowed reads and authorization boundary.
- Compare at least two viable architectures; recommend the smallest JSON/JSONL append-only design and state when to switch.
- Preserve single-run `ExperimentManifest` separately from family lineage.
- Freeze six conceptual objects or justified final equivalents: Family Spec, Trial Lineage, Trial Attempt, Trial Selection, Family Manifest, Validation Result.
- Freeze trial/attempt/seed/duplicate/failure/cancel/exclusion count and state-machine semantics.
- Freeze deterministic canonical serialization/hash, immutable seal, supersession chain and tamper validation.
- Cover CPI-CR163-001..004 as four mappings across two deduplicated producer chains; S03 remains one Story and all five Story outcomes remain unchanged.
- Integrate through existing CR151/CR154/admission package consumers; do not create a new gate family.
- Keep effective-trial unavailable and CR155 blocked; do not overclaim C1 computation.
- Include explicit caller/adjacent contracts: call direction, timing, trigger, inputs, outputs, continuation, fallback and required caller changes.
- Include preconditions/failure behavior, quantitative success criteria, Use Case→Architecture traceability, 3+ scenario simulations, risk matrix, ADRs, rollout and rollback.
- Include a substantive Gotchas section and a revision record.

## Forbidden Work

Do not modify source, tests, DEVELOPMENT-PLAN, Story/LLD files, quality/release docs, runtime state, credentials, data, external systems or Git remote. Do not open the formal CP3 gate.

## Return Contract

Write all outputs listed by the capsule, including the return summary. If a blocking architectural choice remains, return a compact DQ batch with recommendation/alternatives/pros-cons/impact/switch condition for host relay; otherwise mark ready_for_cp3_review=true.
