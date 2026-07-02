---
story_id: "CR152-S01-pit-feature-label-contracts"
change_id: "CR-152"
title: "PIT feature matrix and label policy contracts"
status: "ready-for-verification"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
depends_on: []
implementation_allowed: true
authorization_boundary: "static/fixture only; no real training, real data validation, lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/model registry/catalog pointer/Git remote"
---

# CR152-S01 PIT Feature Matrix and Label Policy Contracts

## Goal

Define the metadata-only PIT feature matrix, label policy and leakage guard contracts for CR152.

## Scope

- PIT feature matrix metadata anchored to `ResearchDatasetSpec`.
- Label policy companion anchored to `LeakagePolicy`.
- `label_method` slots: `fixed_window`, `triple_barrier`, `meta_label`.
- Static validation for feature availability, decision time and label availability.

## Acceptance Criteria

- PIT matrix contract includes entity/date keys, decision time, feature availability, feature schema hash and lineage refs.
- Label policy reserves triple-barrier and meta-label spec slots without implementing their algorithms.
- CP5 LLD makes active `triple_barrier` first-wave enforcement exact; recommended result is `BLOCKED` because the algorithm is not implemented.
- Leakage fixtures include passing and fail-closed examples.
- No existing `ResearchDatasetSpec` or `LeakagePolicy` replacement is introduced.
- No runtime, real data, external framework or registry/store write is authorized.

## File Ownership

| File | Intent |
|---|---|
| `engine/research_production_contracts.py` | PIT feature matrix and label policy companion contracts. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Static contract and leakage fixtures. |

## Design Evidence Required For CP5

Full LLD covering fields, enums, validation helpers, serialization format, triple-barrier inactive enforcement, leakage failure behavior, fixture design and rollback strategy.
