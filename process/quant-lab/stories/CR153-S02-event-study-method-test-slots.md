---
story_id: "CR153-S02-event-study-method-test-slots"
change_id: "CR-153"
title: "Event study method, test family and multiple-testing slots"
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
implementation_allowed: false
authorization_boundary: "static/fixture only; no full statistics implementation, real event feed, real returns validation, provider/lake/runtime/broker/credential/event store/catalog/model registry"
---

# CR153-S02 Event Study Method, Test Family and Multiple-Testing Slots

## Goal

Define event study method evidence, test family slots and EV-GAP-7 multiple testing / data snooping slot.

## Scope

- `EventStudyMethodSpec` fields for estimation window, event window, normal return model and return horizon.
- CAR / BHAR / calendar-time method slots.
- Patell / BMP / generalized sign / rank / bootstrap slots.
- `multiple_testing_or_data_snooping_slot` with required machine-visible fields.

## Acceptance Criteria

- Method evidence cannot collapse event study into ordinary forward return.
- Test family slots include status, refs, sample count, raw p-value and adjusted p-value fields where applicable.
- Multiple testing slot includes `family_id`, `tested_window_count`, `correction_method`, `adjusted_p_value`, `status`, and `report_ref` or `n/a_reason`.
- White Reality Check, Hansen SPA, Romano-Wolf, PBO, DSR and similar algorithms remain slot-only/deferred unless a later CR authorizes implementation.
- Unsupported active method selection fails closed or requires review per CP5.
- Field ownership is limited to method, test family and multiple-testing / data-snooping fields; S02 must not define overlap, cluster, endogeneity, event CV or universe PIT audit fields owned by S03.

## File Ownership

| File | Intent |
|---|---|
| `engine/event_strategy_contracts.py` | Event study method and test report companion contracts. |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | Static method/test/multiple-testing fixtures. |

## Design Evidence Required For CP5

Full LLD covering method enums, slot schema, validation statuses, unsupported-method handling, EV-GAP-7 traceability and fixture cases.

The LLD must include a field partition table for the shared `engine/event_strategy_contracts.py` and `tests/research/test_event_driven_strategy_e2e_contracts.py` surfaces, explicitly marking S03 bias/CV/universe fields as out of scope.
