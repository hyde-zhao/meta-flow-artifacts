---
story_id: "CR155-S03-backtest-oos-walkforward-validation"
cr_id: "CR-155"
title: "Historical backtest and OOS/walk-forward validation flow"
status: "lld-ready-for-review"
priority: "P0"
lld_policy: "full-lld"
feature_design_refs:
  - "docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md"
  - "docs/features/daily-multifactor-baseline-strategy-artifact/TEST-PLAN.md"
depends_on:
  - "CR155-S01-baseline-artifact-contract"
  - "CR155-S02-readonly-data-provenance-adapter"
lld_ref: "process/stories/CR155-S03-backtest-oos-walkforward-validation-LLD.md"
---

# CR155-S03 Backtest/OOS/Walk-Forward Validation

Define the historical backtest and OOS/walk-forward flow that consumes CR148 foundation contracts, including split manifest, purge/embargo policy, metrics, cost/risk/attribution refs and failure statuses.

CP5 must review `process/stories/CR155-S03-backtest-oos-walkforward-validation-LLD.md`. The design must not introduce optimizer/model search, external framework execution or production runtime.
