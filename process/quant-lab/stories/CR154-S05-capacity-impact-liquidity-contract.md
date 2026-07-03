---
story_id: "CR154-S05-capacity-impact-liquidity-contract"
change_id: "CR-154"
title: "Gate 4 capacity, impact and liquidity contract"
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
authorization_boundary: "local/static/fixture-only planning; no real TCA, execution replay, broker fills or live data"
---

# CR154-S05 Gate 4 Capacity, Impact and Liquidity Contract

## Goal

Define capacity, market impact and liquidity sizing evidence contracts without claiming real TCA or execution readiness.

## Scope

- ADV participation, capacity dollars, liquidity sizing refs and cost-underestimation status.
- `impact_model_family` enum.
- No-real-TCA release wording.

## Acceptance Criteria

- `impact_model_family` includes `square_root`, `almgren_chriss`, `gatheral`, `custom`, `n/a-with-reason`.
- `custom` carries rationale, inputs, validation boundary and wording limits.
- Missing capacity/impact evidence blocks production-like claims or follows tier-specific review policy.
- Gate 4 blocked status propagates into Gate 1 impact/capacity refs and blocked claims.
- No real execution, broker, fill, TCA or market data replay is used.

## File Ownership

| File | Intent |
|---|---|
| `engine/cross_strategy_reliability_gates.py` | Gate 4 capacity/impact section. |
| `tests/research/test_cross_strategy_reliability_gates.py` | Gate 4 enum and blocked-claim fixtures. |

## Design Evidence Required For CP5

Full LLD covering enum serialization, validation, threshold/default policy, blocked-claim mapping, fixture cases and no-real-TCA wording.
