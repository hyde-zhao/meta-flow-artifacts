---
story_id: "CR154-S04-pit-universe-survivorship-gate"
change_id: "CR-154"
title: "Gate 3 PIT universe and survivorship gate"
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
authorization_boundary: "local/static/fixture-only planning; no real universe construction, provider fetch or lake read/write"
---

# CR154-S04 Gate 3 PIT Universe and Survivorship Gate

## Goal

Define shared survivorship-free / PIT universe gate semantics and the CR153 `universe_pit_audit` lifecycle.

## Scope

- PIT universe status, source refs, as-of / available-at audit refs and survivorship-free evidence refs.
- Delegation lifecycle for CR153 `universe_pit_audit`.
- Blocked claims for non-PIT or missing universe evidence.

## Acceptance Criteria

- CR153 slot remains as first-wave source evidence and is marked delegated-to-CR154 for shared release wording.
- CR154 first wave does not delete CR153 slot semantics.
- Missing or non-PIT universe evidence blocks survivorship-free and production-like claims.
- No real universe data is built, fetched, read, written or published.

## File Ownership

| File | Intent |
|---|---|
| `engine/cross_strategy_reliability_gates.py` | Gate 3 PIT universe section. |
| `engine/event_strategy_contracts.py` | Compatibility reference only after CP5; no deletion in first wave. |
| `tests/research/test_cross_strategy_reliability_gates.py` | Gate 3 fixture cases. |

## Design Evidence Required For CP5

Full LLD covering CR153 slot lifecycle, source-ref semantics, blocked claims, cross-gate propagation into Gate 1 and no-real-universe boundary.
