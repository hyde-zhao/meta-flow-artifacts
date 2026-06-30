# CR139-S10 LLD - ML Lake Onboard No Bypass

## 1. Story Scope

S10 将实验 15 与实验 23-29 的目标执行入口接入 CR139 只读 lake 输入契约，禁止目标入口继续以 `--data-dir` / `load_local_frames` 作为默认事实源。历史本地 parquet helper 可保留为旧测试和旧实验兼容层，但不得被 S10 目标入口直接调用。

## 2. Inputs

- `experiments/run_experiment_15_factor_framework.py`
- `experiments/run_experiment_23_29_ml_factor_suite.py`
- `market_data.readers.read_panel_as_of`
- Fixture/static tests only; no real lake, provider, catalog, NAS, runtime, QMT, simulation, live, or Git remote writes.

## 3. Output Contract

- New lake input adapter returns typed frames for `prices`, `index_members`, and `trade_calendar`.
- Target entry functions route through that adapter before building matrices.
- Static evidence proves target entry functions do not directly call `load_local_frames` and do not require `--data-dir`.
- Legacy `load_local_frames(Path)` remains callable for non-S10 historical tests and scripts.

## 4. Module Design

Add a small helper module under `experiments/`:

- `ExperimentLakeInputRequest`: dataset request fields (`lake_root`, `as_of`, date range, symbols, quality policy).
- `ExperimentLakeInputResult`: frame mapping, source lineage, adapter status, and no-real-operation counters.
- `load_experiment_lake_frames(args, reader=read_panel_as_of)`: validates explicit lake/as-of request and composes the three required frames through PIT reader calls.

Update target scripts:

- Experiment 15 `parse_args` replaces required `--data-dir` with lake/as-of flags.
- `run_factor_framework` calls `load_experiment_lake_frames`.
- Experiment 23-29 `parse_args` replaces required `--data-dir` with lake/as-of flags.
- `run_stage4_suite` calls `load_experiment_lake_frames`.

## 5. Data Rules

- `prices` must be read PIT as of decision time through `read_panel_as_of`.
- `index_members` and `trade_calendar` are read through the same adapter with fixture reader injection in tests.
- Missing `lake_root` or missing `as_of` returns typed failure, not implicit environment fallback.
- No fallback to repo `data/` or ad hoc parquet directory is allowed in target entry functions.

## 6. Error Handling

- Missing request fields raise `FactorFrameworkError` at the experiment boundary.
- Non-available PIT reader results raise `FactorFrameworkError` containing dataset and issue code.
- Legacy parquet read errors remain unchanged for older callers of `load_local_frames`.

## 7. Test Plan

Add `tests/test_cr139_ml_lake_onboard_no_bypass.py`:

- Static check: target functions in experiment 15 and 23-29 do not call `load_local_frames`.
- Static check: parsers do not require `--data-dir`.
- Fixture check: injected PIT reader receives all required datasets with identical `as_of` and no writes.
- Compatibility check: legacy `load_local_frames` still supports existing tmp_path parquet fixture callers.

Run targeted regression:

- `tests/test_cr139_ml_lake_onboard_no_bypass.py`
- `tests/test_experiment_15_factor_framework.py`
- `tests/test_experiment_23_29_ml_factor_suite.py`

## 8. File Ownership

Owned by S10:

- `experiments/lake_input_contract.py`
- `experiments/run_experiment_15_factor_framework.py`
- `experiments/run_experiment_23_29_ml_factor_suite.py`
- `tests/test_cr139_ml_lake_onboard_no_bypass.py`

No overlap with S12 implementation files except shared test execution.

## 9. No-Real-Operation Boundary

Counters must remain zero for:

- lake write
- catalog publish
- provider fetch
- NAS sync
- QMT operation
- simulation/live trading
- credential read
- Git remote write

## 10. Rollback

Rollback is limited to reverting S10-owned helper/script/test changes. Legacy `load_local_frames` is intentionally preserved, so rollback does not require data migration.

## 11. Acceptance Mapping

- FD-48: target ML experiments load through `read_panel_as_of` contract.
- FD-48: direct `--data-dir/load_local_frames` bypass count in target entry functions is zero.
- Historical experiment tests remain green through the legacy helper compatibility boundary.

## 12. Dependencies

- Requires S06 PIT reader contract.
- Requires S11 feature/label artifact boundary for downstream lineage but does not write feature artifacts.

## 13. Risks

- Legacy tests pass `data_dir`; adapter must provide an explicit compatibility branch only outside target parser defaults.
- Static acceptance must inspect target execution functions rather than unrelated historical helper definitions.

## 14. CP5 Decision

Proceed to CP6 implementation with fixture/static verification. No additional human gate is required because no real runtime or data-write authorization is introduced.
