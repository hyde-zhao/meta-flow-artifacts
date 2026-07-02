---
story_id: "CR151-S01-statistical-report-contracts"
change_id: "CR-151"
title: "Statistical report contracts"
status: "lld-ready-for-review"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "pending-cp5"
depends_on: []
implementation_allowed: false
authorization_boundary: "static/fixture only; no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote"
---

# CR151-S01 Statistical Report Contracts

## Goal

Define the metadata-only Wave A statistical admission report contracts for CR151.

## Scope

- `MultipleTestingReport`
- `RobustFactorStatisticsReport`
- `WalkForwardValidationPlan`
- `BacktestOverfitRiskReport`
- `StrategyAdmissionStatisticalGate`
- JSON-safe serialization and report-level validation issue surface

## Acceptance Criteria

- All five Wave A objects have explicit required fields matching the CP3 HLD.
- Missing required fields produce structured validation issues.
- Contracts are generic enough for future ML/event adapters and do not hard-code multifactor-only semantics where unnecessary.
- No existing `ResearchDatasetSpec` or `BacktestRunSpec` replacement is introduced.
- No runtime or external IO is authorized.

## File Ownership

| File | Intent |
|---|---|
| `engine/strategy_admission_statistical_gate.py` | New contract module skeleton and report dataclasses. |
| `tests/test_cr151_strategy_admission_statistical_gate.py` | Contract serialization and validation fixtures. |

## Design Evidence Required For CP5

Full LLD covering fields, enums, validation helpers, serialization format, failure behavior, test fixtures and migration/rollback strategy.
