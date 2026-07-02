---
story_id: "CR153-S01-event-research-time-pit-contracts"
change_id: "CR-153"
title: "Event research time semantics and PIT revision gate"
status: "lld-ready-for-review"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "pending"
depends_on: []
implementation_allowed: false
authorization_boundary: "static/fixture only; no real event feed, listener, lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/event store/catalog/model registry/Git remote"
---

# CR153-S01 Event Research Time Semantics and PIT Revision Gate

## Goal

Define `EventResearchSpec`, event identity, three-time semantics and event revision/PIT fail-closed gate for CR153.

## Scope

- Event identity, event type, entity key and source snapshot refs.
- `event_occurred_at`, `event_announced_at`, `event_available_at` and `decision_time`.
- Revision policy and availability/PIT validation issues.
- Static fixtures for passing and look-ahead cases.

## Acceptance Criteria

- Time fields are required or explicitly N/A with reason where CP5 permits.
- `event_available_at > decision_time` returns `BLOCKED`.
- Availability is never inferred from occurred or announced time.
- Missing revision policy is represented as `NEEDS_REVIEW` or `BLOCKED` per CP5 design.
- No real lake, NAS, provider, credential, catalog current truth or event feed is read.

## File Ownership

| File | Intent |
|---|---|
| `engine/research_production_contracts.py` | Event research companion fields and PIT/revision validation anchors. |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | Static time semantics and PIT fixtures. |

## Design Evidence Required For CP5

Full LLD covering event fields, validation helpers, status mapping, failure behavior, fixture design, serialization, anchor delta mapping and rollback strategy.
