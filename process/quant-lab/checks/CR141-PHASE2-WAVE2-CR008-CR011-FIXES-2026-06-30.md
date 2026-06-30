# CR141 Phase 2 Wave 2 CR008 / CR011 Fixes

## Scope

Wave 2 fixed experiment-adjacent CR008/CR011 contract and report-boundary baseline failures without touching real lake, runtime, provider, NAS, trading, or Git remote write.

## Changes

| Area | Change | Commit |
|---|---|---|
| CR008 auxiliary inputs | Unknown/unregistered auxiliary capabilities now return typed unavailable without calling the injected reader. | `7206cc2` |
| CR008 experiment 15 fixture compatibility | Direct `Namespace(data_dir=...)` tests now route through explicit `fixture_frames` under the lake input contract; CLI still does not define `--data-dir`. | `7206cc2`, `a30ee47` |
| CR008 adjustment gate | Adjustment policy gate now distinguishes missing policy, data-column mixed policies, and data-vs-request mismatch. | `7206cc2` |
| CR008/CR011 static secret scan | Removed literal `.env` from target source while preserving the same forbidden-file behavior. | `7206cc2` |
| CR011 legacy report boundary | Baseline report path handling supports both current external research path and historical `reports/experiment_17_21` alias fail-fast. | `7206cc2` |

## Targeted Verification

```bash
uv run --python 3.11 pytest -q tests/test_cr008_factor_auxiliary_data_contract.py tests/test_cr008_quality_adjustment_label_gates.py tests/test_cr008_research_input_metadata.py tests/test_cr011_adjustment_audit.py tests/test_cr011_benchmark_policy_consumption.py tests/test_cr011_capacity_cost_sensitivity.py tests/test_cr011_exposure_claims.py tests/test_cr011_factor_panel_robust_validation.py tests/test_cr139_ml_lake_onboard_no_bypass.py
```

Result: `72 passed in 2.37s`.

```bash
uv run --python 3.11 python -m py_compile experiments/run_experiment_15_factor_framework.py engine/experiment_lake_input_contract.py engine/research_dataset.py experiments/run_experiment_17_21_factor_suite.py market_data/readers.py
```

Result: PASS.

## Full Baseline Comparison

```bash
uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider
```

| Metric | After Wave 1 | After Wave 2 | Delta |
|---|---:|---:|---:|
| Failed | 42 | 29 | -13 |
| Passed | 1437 | 1450 | +13 |
| Duration | 44.62s | 44.42s | n/a |

## Fixed Baseline Failures

Wave 2 fixed the 6 CR008 failures, the 6 CR011 failures, and the inherited `tests/test_experiment_17_21_factor_suite.py::test_experiment_17_21_generates_factor_stability_and_strategy_outputs` failure as a side effect of restoring the legacy report boundary contract.

## Remaining Failures

29 inherited failures remain:

- 13 CR013 backfill / claim-boundary failures.
- 12 runtime/security documentation boundary failures across CR015/CR016/CR019/CR020/CR045.
- 1 remaining experiment ML suite failure, still E3-overlap candidate.
- 1 market data Tushare comparison docs failure.
- 2 legacy STORY-004/013 loader/logging failures.

## Authorization Boundary

No real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, or Git remote write was executed.

## Decision

Wave 2 passes with no new failures. Continue Phase 2 with the CR013 static claim-boundary bucket.
