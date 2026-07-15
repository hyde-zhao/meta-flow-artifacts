---
handoff_id: "CR171-CP7-META-QA-CRITICAL-HANDOFF-2026-07-15"
cr_id: "CR-171"
from: "host-orchestrator"
to: "meta-qa-critical"
phase: "verification"
status: "completed"
created_at: "2026-07-15T22:23:38+08:00"
context_ref: "process/context/CP7-CR171.context.json"
dispatch:
  mode: "subagent"
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "CP3-approved runtime-high-risk decision-only architecture requires independent CP7 design and authorization-boundary verification"
  tool_name: "spawn_agent"
  agent_id: "/root/qa_critical_cr171"
  agent_name: "qa_critical_cr171"
  status: "completed"
  completed_at: "2026-07-15T22:34:38+08:00"
---

# CR-171 CP7 Meta-QA Critical Handoff

## Objective

Independently verify CR-171's CP3-approved **design-only** Stage 3 Launch / Real-Lake Entry Decision Gate. Produce auditable verification evidence for the decision contract and explicitly report whether the historical-evidence classification is sufficient for the *current-entry contract*. This is not a real-data activation, computation or runtime verification.

## Read policy and required inputs

Read `process/context/CP7-CR171.context.json` first. Its must-read list is the default contract. Expand full documents only for verification traceability, contract-boundary conflict or human audit and record each expansion.

Required sources include the approved CP3 HLD, ADR, CP3 result, CP4/CP5/CP6 N/A results, the CR summary and state. Do not read `process/archive/**`, data lake/NAS/provider paths, `.env`, credentials, output paths or runtime sessions.

## Verify exactly

1. CP3's 4/4 decisions are approved and HLD/ADR internal consistency holds.
2. HLD §21 covers each residual item with an explicit `OPEN` or `RESOLVED` state, date, owner, closure route and safe effect while open; verify `OI-CR171-006` is genuinely resolved and OI-001..005 are not falsely closed.
3. ADR-001..004 each carry an explicit usable reopen condition. ADR-002/003/004 must not admit a calendar-only workaround, implicit release inference, repair inside CR-171 or inherited CR-170 waiver.
4. Verify the 5/5 future read fields remain `pending_human_authorization`, deny-default remains 6/6, and no read/write/computation/runtime/trading authorization is present.
5. Verify C1/C2/C3=`insufficient`, C4=`incompatible` are static design verdicts only; no producer, runner, aggregate or CR155 promotion path is implied.
6. Write a compact historical Stage 3 revalidation report based only on existing authorized documentation and the approved contract. It must give exactly one current-entry verdict—expected `insufficient_for_current_entry`—plus legacy annotation. It may not repair, backfill, rerun, mutate manifests, inspect a lake, or claim `stage3_entry_ready=true`.
7. Verify E1/E2 waive-state guards are mechanically specified and FU-006 absence blocks the respective admission/exit action.
8. Verify CP4/CP5/CP6 N/A is valid under the no-tool precommitment; no code, tests, Stories, LLD, tool, producer, data or runtime activity occurred.

## Required outputs

- `process/checks/CR171-HISTORICAL-STAGE3-REVALIDATION-REPORT.md`
- `docs/quality/VERIFICATION-REPORT-CR171-STAGE3-LAUNCH.md`
- `docs/quality/TEST-REPORT-CR171-STAGE3-LAUNCH.md`
- `docs/quality/REVIEW-CR171-STAGE3-LAUNCH.md`
- `docs/quality/FIXES-CR171-STAGE3-LAUNCH.md`
- `process/evidence/CR171-CP7-VERIFICATION.index.json`
- `process/returns/CR171-DESIGN-ONLY.CP7.return.json`
- `process/checks/CP7-CR171-STAGE3-LAUNCH-VERIFICATION.result.json`
- `process/handoffs/CR171-CP7-META-QA-RETURN-SUMMARY.md`

## Non-negotiable fences

- No lake/NAS/provider/credential/environment/output path read or write.
- No C1-C4/current runner/aggregate/CR155 computation, runtime, simulation, paper, live, broker or trading action.
- No code, test, schema, Story, LLD, tool implementation, git remote write, publish/deploy, repair/backfill/rerun or manifest rewrite.
- The only allowed historical conclusion is a report verdict; report success never changes `stage3_started`, `stage3_entry_ready`, `real_data_read_authorized` or `real_computation_authorized` from false.

## Completion protocol

Run the return/evidence/result validators applicable to the generated assets. If any mandatory design contract is inconsistent, record `NEEDS_REWORK` and return to CP3; do not silently correct it. Otherwise leave CP7 PASS with residual OPEN items accurately surfaced for CP8. Do not open/approve CP8 yourself; return the evidence package to Host.
