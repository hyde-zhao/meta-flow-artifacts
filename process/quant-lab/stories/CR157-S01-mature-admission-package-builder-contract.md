---
story_id: "CR157-S01-mature-admission-package-builder-contract"
change_id: "CR-157"
title: "Mature admission package builder contract"
status: "verified-with-risk"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/mature-admission-package/DESIGN.md"
  - "docs/features/mature-admission-package/TEST-PLAN.md"
  - "docs/features/mature-admission-package/TASKS.md"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
  approved_at: "2026-07-05T13:31:00+08:00"
  evidence_ref: "process/stories/CR157-S01-mature-admission-package-builder-contract-LLD.md"
depends_on: []
implementation_allowed: true
authorization_boundary: "CP5 approved local source/test implementation only; no LLD approval, implementation, tests implementation, runtime, real data, publish or trading"
---

# CR157-S01 Mature Admission Package Builder Contract

## Goal

Define the mature admission package builder contract for Stage 2 multifactor research exit, using existing static refs from CR150 / CR151 / CR154 and the CR157 evidence index.

## Scope

- Package object model, readiness status, mandatory Stage 2 exit refs and blocked claims.
- Completeness policy for missing, stale, invalid or `n/a-with-reason` refs.
- Forbidden operation counters and package-level fail-closed semantics.
- Deterministic fixture-only builder behavior.

## Acceptance Criteria

- Package can represent FactorSpec, FactorRunSpec, factor panel, label window, evaluation, portfolio/risk, admission package and evidence index refs.
- Missing mandatory refs are `BLOCKED` or explicitly `n/a-with-reason` only where CP5 policy permits.
- Nonzero forbidden operation counters always produce `BLOCKED`.
- Builder does not read real lake/NAS/provider/runtime/credential resources and does not claim Stage 3 runtime readiness.

## File Ownership

| File | Intent |
|---|---|
| `engine/mature_multifactor_framework.py` | Future package builder contract location, subject to CP5 approval. |
| `tests/research/test_mature_multifactor_framework.py` | Future fixture tests, subject to CP5 approval. |

## Design Evidence Required For CP5

Full LLD covering schema, mandatory refs, state transitions, fixture cases, blocked claims, no-runtime counters, compatibility with CR150 / CR151 / CR154 and rollback behavior.

## CP5 Design Evidence

- Evidence type: `full-lld`
- Evidence ref: `process/stories/CR157-S01-mature-admission-package-builder-contract-LLD.md`
- Review status: `ready-for-cp5-review`

## CP5 Approval

- Status: `approved`
- Approved at: `2026-07-05T13:31:00+08:00`
- Design evidence: `process/stories/CR157-S01-mature-admission-package-builder-contract-LLD.md`
- Implementation boundary: local source/test/docs only; no real lake/NAS/provider/credential/QMT/runtime/simulation/live/trading/broker/catalog/store/registry/publish/external framework/Git remote operation.
