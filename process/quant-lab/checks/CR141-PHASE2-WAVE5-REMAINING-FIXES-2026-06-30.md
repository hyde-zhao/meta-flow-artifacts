# CR141 Phase 2 Wave 5 Remaining Baseline Fixes and Final Validation

## Scope

Wave 5 fixed the final four CR141 baseline failures without using real lake, NAS, provider, runtime, trading, credentials, or Git remote write. The ML suite fix only adds an offline deterministic fallback when optional `ml` dependencies are absent; it does not claim full remaining experiment engine convergence.

## Changes

| Area | Change | Commit |
|---|---|---|
| Legacy loader manifest compatibility | Resolved the default `data/manifests/data_prep_manifest.jsonl` path against an explicit fixture `data_dir` when the repo default path is absent. | `3b12fd6` |
| Tushare remediation docs | Added required manual remediation fields, including `index_code`, `date range`, `lake root`, `resume_policy`, `dry_run`, `path`, and `error enum`. | `3b12fd6` |
| ML suite optional dependency fallback | Added `LightweightTabularRegressor` for offline fixture tests when `sklearn` or `lightgbm` optional dependencies are not installed. | `3b12fd6` |

## Targeted Verification

```bash
uv run --python 3.11 python -m py_compile engine/data_loader.py experiments/run_experiment_23_29_ml_factor_suite.py
```

Result: PASS.

```bash
uv run --python 3.11 pytest -q tests/test_story_004_013.py::test_story_004_loader_reads_offline_parquet_and_quality_gate tests/test_story_004_013.py::test_t_logging_minimal_01_cli_diagnostics tests/test_market_data_tushare_comparison.py::test_docs_record_tushare_backfill_required_missing_proxy_and_backtrader_boundaries tests/test_experiment_23_29_ml_factor_suite.py::test_stage4_suite_generates_expected_reports_with_bounded_concurrency
```

Result: `4 passed in 12.50s`.

```bash
uv run --python 3.11 pytest -q tests/test_cr132_process_artifact_hygiene.py
```

Result: `17 passed in 0.09s`.

## Final Full Validation

```bash
uv run --python 3.11 pytest -q
```

Result: `1479 passed in 54.07s`.

| Metric | Phase 0 Baseline | Final | Delta |
|---|---:|---:|---:|
| Failed | 46 | 0 | -46 |
| Passed | 1433 | 1479 | +46 |
| New failures | 0 | 0 | 0 |

## Fixed Baseline Failures

Wave 5 fixed the final 4 failures:

- 1 ML suite optional dependency / offline fixture failure.
- 1 Tushare remediation documentation field failure.
- 2 STORY-004/013 legacy loader/logging fixture manifest failures.

## Claim Boundary

The ML fallback proves the offline Stage 4 fixture test can run without optional `ml` dependencies in the default development environment. It does not complete `FU-CR140-002` remaining experiment engine convergence and does not validate real lake research semantics.

## Authorization Boundary

No real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, credential read, or Git remote write was executed.

## Decision

CR141 red baseline debt is fully closed: all 46 inherited failures are now passing, and final full pytest reports `1479 passed`.
