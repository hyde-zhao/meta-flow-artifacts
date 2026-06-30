# CR141 Phase 2 Wave 3 CR013 Claim-Boundary Fixes

## Scope

Wave 3 fixed CR013 static claim-boundary baseline failures by moving ignored report dependencies into tracked offline fixtures and by documenting the same claim boundary in the user manual. It did not run or validate against a real lake.

## Changes

| Area | Change | Commit |
|---|---|---|
| CR013 ignored report fixtures | Replaced `reports/`-path test dependencies with tracked `tests/fixtures/cr013/...` files. | `326f388` |
| Execution / VWAP boundary | Added offline fixture coverage for execution claim boundary, unsupported register, full-history gap register, and backfill roadmap. | `326f388` |
| User manual boundary | Added CR013 full-history / unsupported claim boundary text to `docs/USER-MANUAL.md`, matching the README boundary. | `326f388` |

## Targeted Verification

```bash
uv run --python 3.11 pytest -q tests/test_cr013_backfill_roadmap_boundaries.py tests/test_cr013_execution_vwap_claim_boundary.py tests/test_cr013_full_history_gap_register.py tests/test_cr013_unsupported_register_claim_boundary.py
```

Result: `14 passed in 0.45s`.

```bash
uv run --python 3.11 python -m py_compile tests/test_cr013_backfill_roadmap_boundaries.py tests/test_cr013_execution_vwap_claim_boundary.py tests/test_cr013_full_history_gap_register.py tests/test_cr013_unsupported_register_claim_boundary.py
```

Result: PASS.

## Hygiene Check

Before committing source changes, full pytest temporarily showed two CR132 hygiene failures because the CR013 fixture files were still uncommitted and classified as unclassified source status. After source commit `326f388`, the hygiene check passed:

```bash
uv run --python 3.11 python scripts/check_process_artifact_hygiene.py --source-root . --process-root process --json
```

Result: `passed=true`, `unclassified=0`.

## Full Baseline Comparison

```bash
uv run --python 3.11 pytest -q
```

| Metric | After Wave 2 | After Wave 3 | Delta |
|---|---:|---:|---:|
| Failed | 29 | 16 | -13 |
| Passed | 1450 | 1463 | +13 |
| Duration | 44.42s | 44.61s | n/a |

## Fixed Baseline Failures

Wave 3 fixed the 13 CR013 backfill / full-history / unsupported / execution VWAP claim-boundary failures.

## Remaining Failures

16 inherited failures remain:

- 12 runtime/security documentation boundary failures across CR015/CR016/CR019/CR020/CR045.
- 1 remaining experiment ML suite failure: `tests/test_experiment_23_29_ml_factor_suite.py::test_stage4_suite_generates_expected_reports_with_bounded_concurrency`.
- 1 market data Tushare comparison docs failure.
- 2 legacy STORY-004/013 loader/logging failures.

## Authorization Boundary

No real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, or Git remote write was executed.

## Decision

Wave 3 passes with no new failures. Continue Phase 2 with runtime/security documentation boundary failures, then handle the remaining non-runtime docs and loader failures. The ML suite failure remains an E3-overlap candidate unless investigation proves it is only dependency or fixture drift.
