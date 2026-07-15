---
handoff_id: "CR171-CP3-META-SE-CRITICAL-HANDOFF-2026-07-15"
cr_id: "CR-171"
from: "host-orchestrator"
to: "meta-se-critical"
phase: "solution-design"
status: "completed"
created_at: "2026-07-15T21:38:59+08:00"
context_ref: "process/context/CP3-CR171.context.json"
dispatch:
  mode: "subagent"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "CP2-approved Stage 3 architecture route, future real-lake authorization boundary and historical-evidence contract"
  tool_name: "spawn_agent"
  agent_id: "/root/se_critical_cr171"
  agent_name: "se_critical_cr171"
  status: "completed"
  completed_at: "2026-07-15T21:54:50+08:00"
---

# CR-171 CP3 Meta-SE Critical Handoff

## Objective

Produce the CP3 design-only package for the user-approved Stage 3 Launch / Real-Lake Entry Decision Gate. The target is a reviewable architecture and authorization/revalidation contract, **not** a data-lake activation or implementation plan.

## Required context

Read the capsule first and stay within its compact policy:

- `process/context/CP3-CR171.context.json`
- `process/checks/CP2-CR171-STAGE3-LAUNCH-SCOPE.result.json`
- `process/checkpoints/CP2-CR171-STAGE3-LAUNCH-SCOPE.md`
- `process/plans/STAGE3-LAUNCH-DECISION-MEMO-2026-07-15.yaml`

Only expand a full document for an allowed, recorded reason. Do not read `process/archive/**`, lake paths, `.env`, credentials or provider configuration.

## Approved CP2 decisions

| Decision | Value | Design consequence |
|---|---|---|
| route | `c1_c4_real_producer` | Design the two-CR separation: CR-171 decision/authorization/revalidation contract; later activation CR for producer mapping, binding, real-computation authorization and run identity. |
| verifier | `event_bounded_waiver` | Preserve two expiry events: before first real-evidence admission PASS/PASS_WITH_RISK decision; before Stage 3 exit-gate start. CR-170 waiver is not inheritable. |
| future read scope | `scoped_research_data_lake_read_only` | CP3 prepares a human-reviewable five-field candidate: `data_release`, `datasets`, `date_range`, `read_identity`, `output_directory`. It is not a permission to read. |

## Required outputs

- `docs/design/HLD-STAGE3-LAUNCH-REAL-LAKE-ENTRY-DECISION-GATE.md`
- `docs/design/ARCHITECTURE-DECISION-STAGE3-LAUNCH-REAL-LAKE-ENTRY-DECISION-GATE.md`
- `process/discussions/CP3-CR171-HLD-DISCUSSION-LOG.md`
- `process/checks/CP3-CR171-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP3-CR171-STAGE3-LAUNCH-HLD-CONSISTENCY.result.json`
- `process/checkpoints/CP3-CR171-STAGE3-LAUNCH-HLD-REVIEW.md`
- `process/checks/CP3-CR171-HUMAN-GATE-LAUNCH-MESSAGE.md`
- `process/handoffs/CR171-CP3-META-SE-RETURN-SUMMARY.md`

## Mandatory design evidence

1. Include Architecture Gray Areas, at least two options, a selection rationale and explicit fallback/reopen conditions.
2. Make a C1–C4 per-producer feasibility matrix against the proposed frozen release schema/PIT/lineage contract. Each verdict must be a design-time `compatible / insufficient / incompatible`; it must cite declared/static facts only and may not inspect real data.
3. Define the selected two-CR boundary and a consumer/owner map. The later activation CR, never CR-171, owns real-data-to-producer mapping, data release/PIT/lineage binding, real computation authorization and run-identity binding.
4. Specify the five-field future read contract as candidate values/selection rules and deny-default classes. If exact values cannot be established without a read, mark them `pending human authorization` rather than inventing values. CP3 must ask the human gate to freeze them; do not access the lake to resolve them.
5. Define historical revalidation as `classification + legacy annotation + report` only. Its only valid current-entry verdicts are `revalidated_for_current_entry`, `insufficient_for_current_entry`, and `incompatible_rework_required`; repair, backfill, rerun, manifest rewrite and remediation are out of scope and must be follow-up work.
6. Include event-bounded FU-006 waiver state machine and prove that either expiry event prevents an admission PASS/PASS_WITH_RISK or Stage 3 exit-gate start.
7. State the CP3 precommitment that revalidation is pure analysis/manual reading and introduces no executable tool. Any proposed executable tool triggers route reassessment; current CP4–CP6 remain N/A.
8. Include scenario simulations for permitted design-only flow, insufficient/incompatible historical evidence, waiver expiry, and denied read/computation boundary.
9. Explicitly preserve `stage3_started=false`, `stage3_entry_ready=false`, `real_computation_authorized=false`, aggregate=`false`, CR155 promotion=`false`.

## Hard fences

- No real lake/NAS/provider/credential/environment read; no output-directory creation or write.
- No real computation, producer execution, runtime, QMT, broker, simulation, paper, live or trading.
- No code, test, Story, LLD, implementation, aggregate orchestration, CR155 promotion, publish/deploy or Git remote write.
- Do not repair the CR-032/010/018/031/033/168 tracking debt. Disclose only where relevant.
- Do not alter historical runs; annotate/document only.

## Completion protocol

Run relevant design/JSON/YAML consistency checks, record all allowed full-document expansion, and leave CP3 at the required human gate. Return a concise evidence summary to the Host; do not advance to CP4/CP5/CP6 or any runtime action.
