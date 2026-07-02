---
story_id: "CR151-S02-gate-evaluator-fail-closed-rules"
change_id: "CR-151"
title: "Gate evaluator and fail-closed rules"
status: "lld-ready-for-review"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
source_hld: "process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "pending-cp5"
depends_on:
  - "CR151-S01-statistical-report-contracts"
implementation_allowed: false
authorization_boundary: "static/fixture only; no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote"
---

# CR151-S02 Gate Evaluator And Fail-Closed Rules

## Goal

Implement the deterministic admission aggregation policy over the S01 report contracts.

## Scope

- Four-state model: `PASS`, `FAIL`, `NEEDS_REVIEW`, `BLOCKED`
- Mandatory evidence policy
- Threshold failure / borderline evidence policy
- Forbidden operation counter policy
- Fixture coverage for all states

## Acceptance Criteria

- Missing mandatory evidence returns `BLOCKED`.
- Nonzero forbidden operation counters return `BLOCKED`.
- Available evidence that fails thresholds returns `FAIL`.
- Borderline evidence returns `NEEDS_REVIEW` only when mandatory evidence exists.
- Full fixture suite proves at least one path per status.

## File Ownership

| File | Intent |
|---|---|
| `engine/strategy_admission_statistical_gate.py` | Gate evaluator and status aggregation. |
| `tests/test_cr151_strategy_admission_statistical_gate.py` | Status and fail-closed fixtures. |

## Design Evidence Required For CP5

Full LLD covering status precedence, threshold defaults, operation counter schema, fixture matrix and fail-closed proof.
