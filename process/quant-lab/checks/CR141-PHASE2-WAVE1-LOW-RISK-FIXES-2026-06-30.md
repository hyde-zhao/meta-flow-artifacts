# CR141 Phase 2 Wave 1 Low-Risk Fixes

## Scope

Wave 1 fixed isolated low-risk baseline failures that do not require real lake, runtime, provider, NAS, trading, or Git remote write.

## Changes

| Area | Change | Commit |
|---|---|---|
| script entrypoint naming | Moved 26 `scripts/cr139_*.py` gate scripts to `scripts/legacy/cr/`, updated archive allow-list, and updated CR139 tests/legacy command strings. | `ba6dec9` |
| market_data import boundary | Moved schema compatibility contract types/functions into `market_data.contracts`; `engine.contracts` now re-exports them for compatibility; `market_data.readers` no longer imports `engine`. | `ba6dec9` |
| CR131 design surface | Archived CR138/CR139 dedicated HLD/ADR files from `process/docs/design/` to `process/archive/design-cr-docs/` and updated archive index. | `ed1691e` |

## Targeted Verification

```bash
uv run --python 3.11 python -m py_compile market_data/contracts.py market_data/readers.py engine/contracts.py scripts/quality/check_script_entrypoints.py scripts/legacy/cr/cr139_*.py tests/test_cr139_gateb_batch0_preflight.py tests/test_cr139_gateb_batch0_duplicate_profile.py
```

Result: PASS.

```bash
uv run --python 3.11 pytest -q tests/test_script_entrypoint_naming.py tests/test_cr131_design_surface.py tests/test_market_data_contracts.py tests/test_cr139_schema_contract_freeze.py tests/test_cr139_gateb_batch0_preflight.py tests/test_cr139_gateb_batch0_duplicate_profile.py
```

Result: `17 passed in 1.36s`.

## Full Baseline Comparison

```bash
uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider
```

| Metric | Phase 0 | After Wave 1 | Delta |
|---|---:|---:|---:|
| Failed | 46 | 42 | -4 |
| Passed | 1433 | 1437 | +4 |
| Duration | 45.46s | 44.62s | n/a |

## Fixed Baseline Failures

- `tests/test_script_entrypoint_naming.py::test_script_entrypoint_naming_guardrail_passes_current_repo`
- `tests/test_cr131_design_surface.py::test_cr131_design_surface_current_docs_pass`
- `tests/test_cr131_design_surface.py::test_cr131_design_surface_cli_json`
- `tests/test_market_data_contracts.py::test_market_data_does_not_import_engine_family`

## Remaining Failures

42 inherited failures remain. The remaining set contains no new failures outside the Phase 0 baseline. The two remaining experiment suite failures (`test_experiment_17_21...`, `test_experiment_23_29...`) remain deferred-to-E3 by default unless proven fixture-only.

## Authorization Boundary

No real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, or Git remote write was executed.

## Decision

Wave 1 passes. Continue Phase 2 with the next low-risk static/contract bucket.
