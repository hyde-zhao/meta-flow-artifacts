# CR155 Daily Multifactor Baseline Strategy Artifact Verification Report

## Scope

| Item | Result | Evidence |
|---|---|---|
| Verification mode | mixed | Code contract tests, process evidence checks, semantic review, real local lake readonly research run. |
| Verified stories | CR155-S01..S05 | `process/checks/CP6-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-IMPLEMENTATION.result.json` |
| Real lake validation | executed | `process/evidence/CR155-REAL-LAKE-VALIDATION-20260704.json` |
| Rerun consistency | PASS | `cr155-real-lake-baseline-rerun-a` and `cr155-real-lake-baseline-rerun-b` core metrics match exactly. |
| Admission result | FAIL / paper_candidate=false | economic_significance and out_of_sample_validation blocked. |
| Non-scope | lake write, catalog pointer mutation, provider fetch, credential read, runtime, trading, broker, publish, production deployment | Operation counters remain zero for forbidden actions. |

## Verification Object Inventory

| Object | Verification Method | Result |
|---|---|---|
| `engine/daily_multifactor_baseline_artifact.py` | Targeted tests, py_compile, code review | PASS |
| `tests/research/test_daily_multifactor_baseline_artifact.py` | Executed with related regression suite | PASS |
| CP6 return/evidence/result | meta-flow return/evidence/result checks | PASS |
| CP5 approval backfill | human-gate and CP5 result checks | PASS |
| Real local governed lake validation | `scripts/research/run_multifactor_strategy_research.py` executed twice against `/home/hyde/data/quant-lab/data-lake` | PASS_WITH_RISK: execution completed, admission blocked |
| Real-data artifact package | `process/evidence/CR155-DAILY-MULTIFACTOR-BASELINE-REAL-LAKE-ARTIFACT-PACKAGE.json` | PASS_WITH_RISK: package complete, `paper_candidate=false` |
| Authorization boundary | Evidence index, operation counters and code review | PASS |

## Real Lake Validation Summary

| Field | Value |
|---|---|
| Lake root | `/home/hyde/data/quant-lab/data-lake` |
| Date range | `2021-01-01..2026-06-26` |
| Runs | `cr155-real-lake-baseline-rerun-a`, `cr155-real-lake-baseline-rerun-b` |
| Factor panel rows | 6,750,809 |
| Valid research rows | 267,186 |
| Portfolio rows | 20,603 |
| Rebalance count | 260 |
| Mean composite RankIC | 0.0564930614211666 |
| Mean net forward return | -0.0009249957670771533 |
| Mean turnover | 0.336568760539169 |
| Statistical admission | FAIL |
| Cross-strategy reliability | NEEDS_REVIEW |
| Paper candidate | false |

## Gate Results

| Gate | Result | Evidence / Reason |
|---|---|---|
| rerun consistency | PASS | Required metrics and admission status match exactly across both runs. |
| economic_significance | blocked | mean_net_return is negative after costs; return_cost_ratio is negative. |
| out_of_sample_validation | blocked | train_mean_return is positive while test_mean_return is negative; decay_ratio is negative. |
| data_bias_audit | PASS | leakage_count=0. |
| time_split_validation | PASS | yearly segments computed and recorded. |
| capacity_impact | PASS_WITH_RISK | ADV/capacity evidence unavailable. |
| tail_risk | PASS_WITH_RISK | max_drawdown risk remains high. |

## Trace Matrix

| Story | Design Contract | Implementation | Verification | Result |
|---|---|---|---|---|
| S01 artifact contract | Required fields, deterministic serialization, overclaim blocking | `DailyMultifactorBaselineArtifact`, `validate_baseline_artifact` | CR155 S01 tests and real artifact package | PASS |
| S02 provenance | Forbidden counters, fixture fallback, no real-data claim on fallback | `ReadonlyDataProvenance`, provenance validators | CR155 S02 tests and real lake operation counters | PASS |
| S03 validation | Backtest refs, split manifest, required metrics | `BaselineValidationSummary`, split/metric validators | Real lake historical/OOS evidence | PASS_WITH_RISK: OOS gate blocked but captured |
| S04 admission | Separate stat/reliability gates, `paper_candidate`, non-authorization | `DailyMultifactorAdmissionPackage` | Real admission package | PASS_WITH_RISK: `paper_candidate=false` |
| S05 rerun/release | Required metric comparison, strict tolerance, wording guardrail | rerun report and wording helpers | Two real lake reruns | PASS |

## Layered Verification Plan

| Layer | Command / Method | Result |
|---|---|---|
| Unit / fixture | `uv run pytest -q tests/research/test_daily_multifactor_baseline_artifact.py` | PASS: 11 passed |
| Related regression | `uv run pytest -q tests/research/test_strategy_admission_statistical_gate.py tests/research/test_cross_strategy_reliability_gates.py tests/research/test_strategy_admission_package.py` | PASS: 24 passed |
| Integrated test subset | Combined CR155 + related tests | PASS: 35 passed |
| Static syntax | `uv run --python 3.11 python -m py_compile engine/daily_multifactor_baseline_artifact.py` | PASS |
| Real lake rerun A | `uv run --python 3.11 python scripts/research/run_multifactor_strategy_research.py --run-id cr155-real-lake-baseline-rerun-a ...` | Completed; admission blocked |
| Real lake rerun B | `uv run --python 3.11 python scripts/research/run_multifactor_strategy_research.py --run-id cr155-real-lake-baseline-rerun-b ...` | Completed; identical core metrics and same blocked admission |
| Process evidence | return-check, evidence-check, cp result-check | PASS after evidence update |
| Human gate consistency | CP8 human-gate check | PASS after decision brief update |
| Whitespace | `git diff --check -- ...` | PASS |

## Issues And Remaining Risk

| ID | Severity | Status | Description | Owner | Next Step |
|---|---|---|---|---|---|
| R-CR155-CP7-REAL-DATA-VALIDATION-NOT-EXECUTED | MEDIUM | resolved | Real local lake historical/OOS/rerun validation was executed twice. | host-orchestrator | Replaced by `I-CR155-REAL-LAKE-ADMISSION-BLOCKED`. |
| I-CR155-REAL-LAKE-ADMISSION-BLOCKED | HIGH | open-for-CP8-decision | Real lake validation completed and rerun metrics are deterministic, but statistical admission failed closed due `economic_significance` and `out_of_sample_validation`; `paper_candidate=false`. | host-orchestrator | CP8 must decide whether to close CR155 as artifact complete but not paper candidate, or keep active for strategy remediation. |

## Decision

CP7 decision: `PASS_WITH_RISK`.

The implementation and real lake validation evidence are complete for the CR155 artifact scope. The resulting strategy artifact is not admitted as a paper candidate: admission package status is `BLOCKED`, `paper_candidate=false`, and release wording must not imply paper/live/runtime/trading/production readiness.
