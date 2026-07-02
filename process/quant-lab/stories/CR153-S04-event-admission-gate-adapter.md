---
story_id: "CR153-S04-event-admission-gate-adapter"
change_id: "CR-153"
title: "Event admission gate and shared status adapter"
status: "lld-ready-for-review"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/runtime-authorization-safety/DESIGN.md"
source_hld: "process/docs/design/HLD-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "pending"
depends_on:
  - "CR153-S01-event-research-time-pit-contracts"
  - "CR153-S02-event-study-method-test-slots"
  - "CR153-S03-event-bias-risk-audit-slots"
implementation_allowed: false
authorization_boundary: "static/fixture only; event gate PASS is not feed, runtime, broker, paper, live or trading readiness"
---

# CR153-S04 Event Admission Gate and Shared Status Adapter

## Goal

Define `EventStrategyAdmissionGate`, fail-closed event evidence evaluation and shared admission status adapter.

## Scope

- Event-specific gate summary and issues.
- Mapping to `PASS / FAIL / NEEDS_REVIEW / BLOCKED`.
- Forbidden operation counters.
- `StrategyAdmissionPackage` event gate linkage.

## Acceptance Criteria

- Missing mandatory PIT, method, test family, multiple-testing or trace evidence returns `BLOCKED`.
- Nonzero forbidden operation counters return `BLOCKED`.
- Known semantic failures return `FAIL`; incomplete reviewable evidence returns `NEEDS_REVIEW`.
- Admission package linkage records gate presence, requirement, status, refs and blocked reasons.
- Event gate PASS never implies runtime, paper, live, broker, real feed or trading readiness.

## File Ownership

| File | Intent |
|---|---|
| `engine/event_strategy_admission_gate.py` | Event-specific admission gate and adapter contract. |
| `engine/strategy_admission_package.py` | Event gate summary linkage to admission package. |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | Four-state adapter and forbidden-counter fixtures. |

## Design Evidence Required For CP5

Full LLD covering gate fields, adapter mapping, package delta, forbidden operation counters, failure paths and fixture matrix.
