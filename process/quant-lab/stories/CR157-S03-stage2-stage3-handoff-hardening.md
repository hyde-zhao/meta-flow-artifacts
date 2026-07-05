---
story_id: "CR157-S03-stage2-stage3-handoff-hardening"
change_id: "CR-157"
title: "Stage 2 to Stage 3 handoff hardening"
status: "verified-with-risk"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/stage-handoff-guardrails/DESIGN.md"
  - "docs/features/stage-handoff-guardrails/TEST-PLAN.md"
  - "docs/features/stage-handoff-guardrails/TASKS.md"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
  approved_at: "2026-07-05T13:31:00+08:00"
  evidence_ref: "process/stories/CR157-S03-stage2-stage3-handoff-hardening-LLD.md"
depends_on:
  - "CR157-S01-mature-admission-package-builder-contract"
  - "CR157-S02-research-evidence-index-traceability"
implementation_allowed: true
authorization_boundary: "CP5 approved local source/test implementation only; fail-closed handoff; no runtime, real data, publish or trading"
---

# CR157-S03 Stage 2 to Stage 3 Handoff Hardening

## Goal

Define the Stage 2 exit to Stage 3 candidate handoff contract so incomplete or blocked mature strategy candidates cannot be treated as ready.

## Scope

- Handoff object fields: package ref, evidence index ref, readiness status, blocked claims and no-runtime counters.
- Fail-closed behavior for missing package, missing evidence index, unknown status and stale evidence.
- Stage 3 consumer contract: complete package or blocked-with-reason only.

## Acceptance Criteria

- Complete package plus complete evidence index can produce a candidate handoff.
- Missing package or missing evidence index blocks handoff.
- Unknown or absent readiness never defaults to PASS.
- Blocked package carries explicit reasons into Stage 3 candidate intake.

## File Ownership

| File | Intent |
|---|---|
| `engine/mature_multifactor_framework.py` | Future handoff object and validation location, subject to CP5 approval. |
| `tests/research/test_mature_multifactor_framework.py` | Future handoff negative fixture tests, subject to CP5 approval. |

## Design Evidence Required For CP5

Full LLD covering handoff schema, state transitions, consumer contract, blocked claim propagation, failure behavior and rollback/compatibility.

## CP5 Design Evidence

- Evidence type: `full-lld`
- Evidence ref: `process/stories/CR157-S03-stage2-stage3-handoff-hardening-LLD.md`
- Review status: `ready-for-cp5-review`

## CP5 Approval

- Status: `approved`
- Approved at: `2026-07-05T13:31:00+08:00`
- Design evidence: `process/stories/CR157-S03-stage2-stage3-handoff-hardening-LLD.md`
- Implementation boundary: local source/test/docs only; no real lake/NAS/provider/credential/QMT/runtime/simulation/live/trading/broker/catalog/store/registry/publish/external framework/Git remote operation.
