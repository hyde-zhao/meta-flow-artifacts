---
status: "cp4-pass"
version: "0.1"
cr_id: "CR-155"
title: "CR155 Daily Multifactor Baseline Strategy Artifact Story Backlog"
source_hld: "docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md"
feature_design_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md#cr155-cp4-增量daily-multifactor-baseline-strategy-artifact"
development_plan: "process/DEVELOPMENT-PLAN-CR155.yaml"
---

# CR155 Story Backlog

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-04 | host-orchestrator | Initial CP4 Story backlog for CR155 after CP3 approval. |

## Story Overview

| Story ID | Title | Priority | lld_policy | Wave | Depends On | Status |
|---|---|---|---|---|---|---|
| CR155-S01-baseline-artifact-contract | Baseline artifact contract and claim boundary | P0 | full-lld | W1 | - | lld-ready |
| CR155-S02-readonly-data-provenance-adapter | Readonly data provenance adapter and guardrails | P0 | full-lld | W2 | S01 | lld-ready |
| CR155-S03-backtest-oos-walkforward-validation | Historical backtest and OOS/walk-forward validation flow | P0 | full-lld | W2 | S01, S02 | lld-ready |
| CR155-S04-admission-gate-composition-package | Statistical/reliability gate composition and admission package | P0 | full-lld | W3 | S01, S03 | lld-ready |
| CR155-S05-rerun-consistency-release-evidence | Rerun consistency and release evidence wording | P0 | full-lld | W4 | S03, S04 | lld-ready |

## Dependency DAG

```text
CR155-S01
  -> CR155-S02
       -> CR155-S03
            -> CR155-S04
                 -> CR155-S05
CR155-S01 -> CR155-S03
CR155-S01 -> CR155-S04
CR155-S03 -> CR155-S05
```

No cycles or invalid references are present.

## File Ownership Summary

| Future Surface | Primary Story | Notes |
|---|---|---|
| Strategy artifact contract | S01 | Candidate implementation surface: `engine/daily_multifactor_baseline_artifact.py`. |
| Readonly provenance and guardrails | S02 | Must prove forbidden operation counters and no mutation. |
| Backtest/OOS/walk-forward composition | S03 | Must consume CR148 contracts without replacing them. |
| Admission package composition | S04 | Must consume CR151 and CR154 statuses separately. |
| Rerun and release evidence | S05 | Must compare two runs and block overclaim wording. |
| Tests | S01-S05 | Candidate test surface: `tests/research/test_daily_multifactor_baseline_artifact.py`; CP5 may split if needed. |

## Acceptance Summary

| Story | Acceptance Focus |
|---|---|
| S01 | Artifact fields, required refs, non-optimal baseline wording, missing-field failure behavior. |
| S02 | Readonly provenance proof, forbidden operation counters, downgrade to fixture/static when provenance cannot be proven. |
| S03 | Historical backtest, OOS/walk-forward split/fold refs, CR148 report refs, no optimizer/external runtime. |
| S04 | Statistical gate, reliability gate and final package decisions remain separately auditable. |
| S05 | Two rerun metrics are compared and release wording states research artifact only. |

## Not Authorized

CP4 does not authorize LLD approval, source implementation, test implementation, real lake write, catalog pointer mutation, NAS/provider/credential access, QMT/MiniQMT/xtquant runtime, simulation, paper/live/trading, broker operation, external framework run, store/registry write, publish or production deployment.
