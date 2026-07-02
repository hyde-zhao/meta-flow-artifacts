---
story_id: "CR152-S02-purged-embargo-cv-fixture-contract"
change_id: "CR-152"
title: "Purged embargo CV and fixture contract"
status: "ready-for-verification"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
source_hld: "process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
depends_on:
  - "CR152-S01-pit-feature-label-contracts"
implementation_allowed: true
authorization_boundary: "static/fixture only; no real training, real data validation, lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/model registry/catalog pointer/Git remote"
---

# CR152-S02 Purged Embargo CV and Fixture Contract

## Goal

Define purged + embargo CV split policy, split audit and deterministic fixture requirements.

## Scope

- Purged/embargo split policy metadata.
- Split audit for fold overlap, purge window and embargo gap.
- Deterministic fixture schema that can prove positive and negative cases.

## Acceptance Criteria

- Split policy includes fold boundaries, train/validation/test date windows, purge window, embargo days and label horizon.
- Fixture contract includes entity/date, decision_time, feature_available_at, label window, fold boundary, purge window, embargo gap and prediction_timestamp.
- Positive and negative fixture cases are specified for CP6 tests.
- No schema-only fixture that fails to exercise CV behavior is accepted.
- No real lake/provider/NAS/external framework data is read.

## File Ownership

| File | Intent |
|---|---|
| `engine/research_production_contracts.py` | Purged/embargo CV companion contracts and split audit. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Purged/embargo fixture cases. |

## Design Evidence Required For CP5

Full LLD covering split schema, audit algorithm boundaries, fixture size, failure paths, test cases and no-real-data guardrails.
