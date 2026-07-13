---
handoff_id: "CR046-CP4-STORY-PLANNING-META-SE"
from_agent: "host-orchestrator"
to_agent: "meta-se"
status: "returned"
workflow_id: "meta-flow-self-dev"
cr_id: "CR-046"
checkpoint: "CP4"
context_ref: "process/context/CP4-CR046-STORY-PLANNING-CONTEXT.yaml"
---

# CR-046 CP4 Story Planning Handoff

CP3-R3 is approved. Produce CP4 planning/design artifacts only; do not implement code or create Story LLDs.

Required work:

1. Create a Feature Design Matrix for FEAT-EI-CORE/GOVERNANCE/OBSERVABILITY/CORRECTION and required Feature DESIGN/TEST-PLAN/TASKS artifacts in the artifact repository.
2. Create/update `process/DEVELOPMENT-PLAN.yaml` as the seven-Story machine truth source and create seven Story cards with three-part context, `feature_design_refs`, `lld_policy`, dependencies, file ownership, merge owner, TASK IDs and CP5 evidence paths.
3. Preserve the approved DAG and calculate safe LLD/dev/QA waves.
4. Make all seven Stories `full-lld` unless a lower level is strongly justified; platform contract, migration, state, correction, audit and telemetry risks normally require full LLD.
5. Add the three approved minor requirements as machine-checkable CP4/CP5 acceptance:
   - D0 discovery freshness/TTL/session/config-change invalidation and re-probe fixture;
   - followup without reuse receipt cannot inherit verification, with explicit negative fixture;
   - legacy `codex_agent_name` is D3 `self-declared-unverifiable` in strict replay and ST-EI-007 migration.
6. Produce a CP4 result/summary and return summary. Do not open or approve CP5; Host will independently validate and broker the CP5 gate.

Dispatch truth ceiling: this followup reuses a session-observed generic thread. Requested role is meta-se; resolved custom profile/model and reuse receipt are unavailable. Do not claim platform-attested identity.

## Dispatch

- tool: `followup_task`
- thread: `/root/se_cr046_r3_contract`
- dispatch_id: `DISPATCH-CR046-CP4-STORY-PLANNING-SE`
- attempt_id: `ATTEMPT-CR046-CP4-STORY-PLANNING-01`
- requested role: `meta-se`
- resolved custom profile/model: unavailable
- reuse receipt: unavailable
- evidence ceiling: `session-observed/unattested`
