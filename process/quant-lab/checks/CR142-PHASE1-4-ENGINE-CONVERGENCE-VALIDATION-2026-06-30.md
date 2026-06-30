# CR142 Phase 1-4 Engine Convergence Validation

## Scope

CR142 implemented the remaining safe source-only experiment engine convergence slice. It fixed the engine extension contract for custom factor definitions/calculators/direction, then adapted experiment 17_21 group-return evaluation to use `engine.factor_statistics`. Experiment 23_29 consumes the same 17_21 helpers and passed without additional changes.

This validation did not execute or authorize real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, credential read, or Git remote write.

## Changes

| Area | Change | Commit |
|---|---|---|
| Engine extension contract | Added `additional_definitions` to `compute_equity_factor_matrices` so custom calculator registry factors can carry explicit direction metadata. | `8535edd` |
| Contract test | Extended custom calculator test with a negative-direction custom definition and asserted directional matrix equals negative raw matrix. | `8535edd` |
| Experiment 17_21 adapter | Replaced local forward-return calculation with `build_forward_return_matrix`; replaced local group-return loop with `single_sort_returns` adapter. | `8535edd` |
| Experiment 23_29 downstream | No direct code change needed; it imports the 17_21 helpers and passed targeted regression. | `8535edd` |

## Targeted Verification

```bash
uv run --python 3.11 python -m py_compile engine/factor_calculators.py experiments/run_experiment_17_21_factor_suite.py tests/test_factor_calculators.py
```

Result: PASS.

```bash
uv run --python 3.11 pytest -q tests/test_factor_calculators.py tests/test_factor_library.py tests/test_factor_registry.py
```

Result: `13 passed in 0.54s`.

```bash
uv run --python 3.11 pytest -q tests/test_factor_calculators.py tests/test_factor_statistics.py tests/test_experiment_17_21_factor_suite.py tests/test_experiment_23_29_ml_factor_suite.py
```

Result: `10 passed in 16.13s`.

```bash
uv run --python 3.11 pytest -q tests/test_cr132_process_artifact_hygiene.py
```

Result: `17 passed in 0.10s`.

## Full Validation

```bash
uv run --python 3.11 pytest -q
```

Result: `1479 passed in 54.20s`.

## Accepted Boundaries

- `calculate_ic_timeseries` and `summarize_ic` remain experiment-level because `engine.factor_statistics` currently has no equivalent IC summary contract.
- Experiment-level reporting, strategy construction, cost/capacity checks, and fixture orchestration remain in `experiments/`.
- Runner-local stage3 factors are not promoted to equity core factors.

## Decision

CR142 is ready to close as `closed-current-delivery / ready`: the safe remaining experiment engine convergence slice is complete, all tests pass, and no unauthorized operation was executed.
