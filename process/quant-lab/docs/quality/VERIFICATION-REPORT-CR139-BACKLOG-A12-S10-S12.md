---
change_id: "CR-139"
batch_id: "CR139-BACKLOG-A12"
stories: ["CR139-S10", "CR139-S12"]
stage: "CP7"
status: "PASS_WITH_ENV_NOTE"
verified_at: "2026-06-30T19:20:00+08:00"
verified_by: "host-orchestrator-inline"
validation_mode: "static-fixture"
---

# CR139 Backlog-A12 S10/S12 Verification Report

## Scope

| Story | Scope | Non-scope |
|---|---|---|
| CR139-S10 | ML experiment target entries use lake/as-of adapter and no direct local parquet bypass. | Real lake read execution, provider fetch, catalog/pointer write, NAS/runtime/QMT/trading. |
| CR139-S12 | CR139 experiment manifest closure contract with snapshot/as_of/split/model lineage and S29 correctness standard. | Catalog publish, lake manifest/pointer write, model runtime execution. |

## Traceability

| Requirement | Story | Design Contract | Implementation | Verification | Result |
|---|---|---|---|---|---|
| FD-48 | S10 | Target entries call `read_panel_as_of` adapter, not local parquet bypass. | `experiments/lake_input_contract.py`, target scripts | `tests/test_cr139_ml_lake_onboard_no_bypass.py` | PASS |
| FD-48 | S10 | Parser no longer exposes `--data-dir`. | `add_experiment_lake_args()` | `test_parsers_no_longer_define_data_dir` | PASS |
| E1 | S12 | Closure references published snapshot, as_of, split, and lineage. | `ExperimentManifestClosure` | `test_cr139_manifest_closure_records_snapshot_split_model_and_s29_standard` | PASS |
| E2 | S12 | Model artifact hash references dataset snapshot hash. | `validate_experiment_manifest_closure()` | `test_cr139_manifest_closure_fails_when_model_hash_references_different_snapshot` | PASS |
| E5 | S12 | Split cutoff, embargo, and label horizon unified. | `ExperimentSplitPolicy` | S12 closure tests | PASS |
| S29 overlap | S12 | Closure requires S29 correctness standard ref. | `s29_correctness_standard_ref` | `test_cr139_manifest_closure_fails_when_s29_standard_is_missing` | PASS |
| CR030 compatibility | S12 | Existing manifest/catalog tests remain green. | Additive API only | `tests/test_cr030_experiment_manifest_catalog.py` | PASS |

## Commands

| Command | Result |
|---|---|
| `uv run --python 3.11 python -m py_compile experiments/lake_input_contract.py experiments/run_experiment_15_factor_framework.py experiments/run_experiment_23_29_ml_factor_suite.py engine/research_manifest.py tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_experiment_manifest_closure.py tests/test_experiment_15_factor_framework.py tests/test_experiment_23_29_ml_factor_suite.py` | PASS |
| `uv run --python 3.11 pytest -q tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_experiment_manifest_closure.py tests/test_experiment_15_factor_framework.py tests/test_cr030_experiment_manifest_catalog.py` | PASS, 18 passed in 0.45s |
| `uv run --python 3.11 pytest -q tests/test_experiment_23_29_ml_factor_suite.py` | ENV-BLOCKED, `ModuleNotFoundError: No module named 'sklearn'` before model training |

## Boundary Verification

No provider fetch, credential read, real lake write, active catalog/manifest write, pointer advance, physical migration, NAS operation, QMT/runtime/trading operation, dependency install, or Git remote write was performed.

## Findings And Risks

No S10/S12 contract findings. One environment note remains: the legacy Stage4 end-to-end regression requires optional `sklearn` in this environment. S10 acceptance is still covered by static no-bypass checks and adapter fixture tests.

## Stage Decision

Decision: `PASS_WITH_ENV_NOTE`. S10 and S12 can move to `verified`.
