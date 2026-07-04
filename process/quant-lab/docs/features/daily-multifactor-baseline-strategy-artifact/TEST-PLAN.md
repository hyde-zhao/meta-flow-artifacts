---
status: "cp4-planned"
version: "0.1"
feature_id: "FEAT-16"
cr_id: "CR-155"
title: "Daily Multifactor Baseline Strategy Artifact Test Plan"
---

# Daily Multifactor Baseline Strategy Artifact Test Plan

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-04 | host-orchestrator | Initial CP4 test plan for CR155 design evidence preparation. |

## Validation Layers

| Layer | Coverage |
|---|---|
| Contract tests | Required artifact fields, JSON-safe serialization, missing mandatory refs, status enum handling. |
| Safety tests | Readonly provenance, forbidden operation counters, no credential/env/NAS/provider/runtime/trading/store/catalog writes. |
| Backtest/OOS tests | Historical backtest refs, split manifest, purge/embargo, walk-forward fold integrity, metric presence. |
| Admission tests | Statistical gate status, reliability gate status, final admission package status and reasons remain separate. |
| Rerun tests | Two-run metric comparison over return, drawdown, turnover, costs, liquidity/capacity summary and admission status. |
| Release wording tests | Evidence states research artifact only; no paper/live/runtime/trading/production readiness claim. |

## Required CP7 Evidence Families

| Evidence Family | Required Result |
|---|---|
| Artifact schema validation | PASS or BLOCKED with missing-field reasons. |
| Readonly boundary validation | PASS only if forbidden operation counters remain zero. |
| Historical backtest validation | PASS/FAIL/NEEDS_REVIEW with metric refs. |
| OOS/walk-forward validation | PASS/FAIL/NEEDS_REVIEW with split/fold refs. |
| Statistical admission | PASS/FAIL/NEEDS_REVIEW from CR151-compatible evaluator. |
| Reliability admission | PASS/FAIL/NEEDS_REVIEW from CR154-compatible evaluator. |
| Admission package | `paper_candidate=true|false` plus structured reasons. |
| Rerun consistency | PASS only when required metric diffs are within CP5-defined tolerance. |

## Non-Authorized Test Activity

The test plan does not authorize lake write, NAS sync, provider fetch, credential read, QMT/MiniQMT/xtquant runtime, simulation, paper/live trading, broker access, external framework run, catalog pointer mutation, store/registry writes, publish or production deployment.
