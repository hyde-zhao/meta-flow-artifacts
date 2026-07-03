---
story_id: "CR154-S03-cross-strategy-cv-governance"
change_id: "CR-154"
title: "Gate 2 cross-strategy CV governance"
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
authorization_boundary: "local/static/fixture-only planning; no real training, runtime or provider data"
---

# CR154-S03 Gate 2 Cross-Strategy CV Governance

## Goal

Define shared walk-forward / OOS / purged-embargo governance across multifactor, ML and event-driven strategies.

## Scope

- Split manifest refs and OOS split refs.
- Purge / embargo refs and leakage status.
- Strategy-specific adapter mapping without forcing ML-only semantics onto non-ML strategies.

## Acceptance Criteria

- Shared CV contract works for multifactor, ML and event-driven strategies.
- CR152 ML CV evidence can be consumed as one adapter input.
- Missing OOS or purge/embargo evidence fails closed or produces explicit `n/a-with-reason` only where tier policy allows.
- Fixture cases cover pass, missing OOS, missing purge/embargo and strategy-specific n/a.

## File Ownership

| File | Intent |
|---|---|
| `engine/cross_strategy_reliability_gates.py` | Gate 2 CV governance section. |
| `tests/research/test_cross_strategy_reliability_gates.py` | Gate 2 fixture cases. |

## Design Evidence Required For CP5

Full LLD covering shared CV fields, adapter mapping, fail-closed behavior, fixture matrix and relationship to CR152 ML-specific CV fields.
