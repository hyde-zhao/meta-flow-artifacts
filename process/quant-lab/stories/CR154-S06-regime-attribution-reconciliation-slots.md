---
story_id: "CR154-S06-regime-attribution-reconciliation-slots"
change_id: "CR-154"
title: "Gate 5 regime, attribution and reconciliation slots"
status: "dev-ready"
owner: "meta-dev"
feature_design_refs:
  - "process/docs/features/cross-strategy-reliability-gates/DESIGN.md"
  - "process/docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"
  - "process/docs/features/cross-strategy-reliability-gates/TASKS.md"
source_hld: "process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-03T06:58:00+08:00"
depends_on:
  - "CR154-S01-shared-gate-contract-fixture-skeleton"
implementation_allowed: true
authorization_boundary: "local/static/fixture-only planning; no runtime, broker or real reconciliation"
---

# CR154-S06 Gate 5 Regime, Attribution and Reconciliation Slots

## Goal

Make Gate 5 explicit as a first-wave Story with structured slot/status/ref/n/a semantics.

## Scope

- Regime evidence slots.
- Attribution evidence slots.
- Reconciliation evidence slots.
- `n/a-with-reason` and release wording limits.

## Acceptance Criteria

- Gate 5 is not hidden inside Phase D compatibility work.
- Slot omission without reason is invalid.
- Missing slot evidence maps to `NEEDS_REVIEW` or `BLOCKED` per tier policy.
- Release wording cannot imply real runtime, broker, paper/live or reconciliation readiness.
- No real reconciliation, broker query, account data, live feed or runtime action is performed.

## File Ownership

| File | Intent |
|---|---|
| `engine/cross_strategy_reliability_gates.py` | Gate 5 slot section. |
| `tests/research/test_cross_strategy_reliability_gates.py` | Gate 5 slot and wording fixtures. |

## Design Evidence Required For CP5

Full LLD covering slot schema, status mapping, n/a reason validation, tier interaction and no-runtime reconciliation boundary.
