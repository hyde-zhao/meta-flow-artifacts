---
story_id: "CR154-S07-admission-default-policy-tier-resolution"
change_id: "CR-154"
title: "Gate 6 admission default policy tier resolution"
status: "dev-ready"
owner: "meta-dev"
feature_design_refs:
  - "process/docs/features/cross-strategy-reliability-gates/DESIGN.md"
  - "process/docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"
  - "process/docs/features/runtime-authorization-safety/DESIGN.md"
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
  - "CR154-S02-statistical-artifacts-and-trap-severity"
  - "CR154-S03-cross-strategy-cv-governance"
  - "CR154-S04-pit-universe-survivorship-gate"
  - "CR154-S05-capacity-impact-liquidity-contract"
  - "CR154-S06-regime-attribution-reconciliation-slots"
implementation_allowed: true
authorization_boundary: "local/static/fixture-only planning; no runtime authorization or release execution"
---

# CR154-S07 Gate 6 Admission Default Policy Tier Resolution

## Goal

Define the tier resolver that turns gate evidence into opt-in, default-required, release-blocking or not-authorized release wording.

## Scope

- HLD §8 T0/T1/T2/T3 policy table.
- Resolver form: config mapping, function or hybrid.
- Strategy-class overrides and fallback rules.
- Unknown profile fail-closed behavior.

## Acceptance Criteria

- Tier resolution is explicit enough for CP5 review.
- T0 exploratory, T1 admission candidate, T2 production-like/release-readiness and T3 paper/live/trading/runtime profiles are covered.
- Unknown release profile fails closed until classified.
- Gate PASS cannot imply paper/live/trading/broker readiness.
- Existing CR151/CR152/CR153 package-visible behavior has compatibility fixtures.

## File Ownership

| File | Intent |
|---|---|
| `engine/cross_strategy_reliability_gates.py` | Tier resolver and policy output section. |
| `engine/strategy_admission_package.py` | Future summary attachment only after CP5 approval. |
| `tests/research/test_cross_strategy_reliability_gates.py` | Tier fixtures. |
| `tests/research/test_strategy_admission_package.py` | Admission package compatibility fixtures. |

## Design Evidence Required For CP5

Full LLD covering resolver signature, config ownership, fallback behavior, release wording, compatibility tests, rollback and no-runtime authorization boundary.
