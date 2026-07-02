---
story_id: "CR153-S03-event-bias-risk-audit-slots"
change_id: "CR-153"
title: "Cluster, endogeneity, CV and universe PIT audit slots"
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
depends_on:
  - "CR153-S01-event-research-time-pit-contracts"
  - "CR153-S02-event-study-method-test-slots"
implementation_allowed: false
authorization_boundary: "static/fixture slots only; full CV, survivorship, capacity, impact, regime and reconciliation governance deferred to CR154"
---

# CR153-S03 Cluster, Endogeneity, CV and Universe PIT Audit Slots

## Goal

Keep event bias and reliability risks machine-visible while preserving CR153 first-wave scope.

## Scope

- Overlap / cluster report slot.
- Endogeneity / self-selection treatment slot.
- Event CV split audit refs.
- Universe PIT audit slot.
- Deferred refs for capacity, impact, regime and reconciliation.

## Acceptance Criteria

- Cluster and overlap evidence is structured with status, refs and `n/a_reason`.
- Endogeneity treatment is structured with status, refs and `n/a_reason`.
- Event CV slot records split audit refs without implementing full walk-forward or purged-embargo governance.
- Universe PIT audit slot is visible without claiming survivorship-free universe gate completion.
- Deferred CR154 risks cannot be silently marked complete.
- Field ownership is limited to overlap, cluster, endogeneity, event CV and universe PIT audit fields; S03 must not redefine S02 method, test family or multiple-testing / data-snooping fields.

## File Ownership

| File | Intent |
|---|---|
| `engine/event_strategy_contracts.py` | Bias risk and deferred governance slot contracts. |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | Static slot validation fixtures. |

## Design Evidence Required For CP5

Full LLD covering slot schema, status semantics, CR154 handoff refs, fail-closed / needs-review behavior and fixture validation.

The LLD must include a field partition table for the shared `engine/event_strategy_contracts.py` and `tests/research/test_event_driven_strategy_e2e_contracts.py` surfaces, explicitly marking S02 method/test/multiple-testing fields as read-only inputs.
