---
change_id: "CR-139"
batch_id: "CR139-BACKLOG-A12"
stories: ["CR139-S10", "CR139-S12"]
stage: "CP6"
status: "complete"
implemented_at: "2026-06-30T19:10:00+08:00"
implemented_by: "host-orchestrator-inline"
validation_mode: "static-fixture"
---

# CR139 Backlog-A12 S10/S12 Implementation

## Scope

| Story | Implemented | Non-scope |
|---|---|---|
| S10 | Experiment lake input adapter, target parser lake/as-of flags, target entry functions no direct local parquet bypass, explicit fixture-frame test injection. | Real lake read execution, provider fetch, catalog publish, NAS sync, QMT/runtime/trading, dependency install. |
| S12 | CR139 experiment manifest closure dataclasses/build/validate helpers with S29 standard reference, snapshot/split/model lineage closure, and fail-closed validation. | Catalog publishing, lake manifest/pointer writes, model runtime execution, artifact file writes. |

## Implementation Objects

| File | Change |
|---|---|
| `experiments/lake_input_contract.py` | Added CR139 experiment lake input request/result, parser args helper, and `read_panel_as_of` adapter with fixture injection. |
| `experiments/run_experiment_15_factor_framework.py` | Replaced target parser `--data-dir` with lake/as-of args and routed `run_factor_framework` through lake input contract. |
| `experiments/run_experiment_23_29_ml_factor_suite.py` | Replaced target parser `--data-dir` with lake/as-of args and routed `run_stage4_suite` through lake input contract. |
| `engine/research_manifest.py` | Added CR139 closure constants, snapshot/split/model dataclasses, builder, and validator. |
| `tests/test_cr139_ml_lake_onboard_no_bypass.py` | Added S10 static/fixture contract tests. |
| `tests/test_cr139_experiment_manifest_closure.py` | Added S12 E1/E2/E5 and S29-reference tests. |
| `tests/test_experiment_15_factor_framework.py` | Updated legacy fixture construction to explicit fixture-frame injection. |
| `tests/test_experiment_23_29_ml_factor_suite.py` | Updated legacy fixture construction to explicit fixture-frame injection. |

## Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| ML target entries must not directly call local parquet bypass. | `run_factor_framework()` and `run_stage4_suite()` call `load_experiment_lake_frames()`. | `test_target_entry_functions_do_not_call_local_parquet_bypass` |
| Target parser must not expose required `--data-dir`. | `add_experiment_lake_args()` adds `--lake-root` and `--as-of`. | `test_parsers_no_longer_define_data_dir` |
| PIT input must use `read_panel_as_of` contract. | `load_experiment_lake_frames()` invokes reader for required datasets with identical as_of. | `test_lake_input_contract_calls_pit_reader_for_required_datasets` |
| Manifest closure must reference snapshot/as_of/split/lineage. | `ExperimentManifestClosure` with nested snapshot/split/model refs. | `test_cr139_manifest_closure_records_snapshot_split_model_and_s29_standard` |
| Model artifact hash must bind to dataset snapshot hash. | `validate_experiment_manifest_closure()` mismatch check. | `test_cr139_manifest_closure_fails_when_model_hash_references_different_snapshot` |
| S12 must use S29 correctness standard. | Required `s29_correctness_standard_ref`. | `test_cr139_manifest_closure_fails_when_s29_standard_is_missing` |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 python -m py_compile experiments/lake_input_contract.py experiments/run_experiment_15_factor_framework.py experiments/run_experiment_23_29_ml_factor_suite.py engine/research_manifest.py tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_experiment_manifest_closure.py tests/test_experiment_15_factor_framework.py tests/test_experiment_23_29_ml_factor_suite.py` | PASS |
| `uv run --python 3.11 pytest -q tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_experiment_manifest_closure.py tests/test_experiment_15_factor_framework.py tests/test_cr030_experiment_manifest_catalog.py` | PASS, 18 passed in 0.45s |
| `uv run --python 3.11 pytest -q tests/test_experiment_23_29_ml_factor_suite.py` | ENV-BLOCKED, optional `sklearn` dependency missing before model training; S10 static/adapter contract already verified. |

## Boundary Check

No provider fetch, credential read, real lake write, active catalog/manifest write, pointer advance, physical migration, NAS operation, broker/QMT/runtime/trading operation, dependency install, or Git remote write was performed. Fixture writes are limited to pytest `tmp_path` in legacy tests.
