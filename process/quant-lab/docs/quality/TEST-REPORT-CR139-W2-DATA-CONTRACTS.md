# CR139 W2 Data Contracts Test Report

## Result

| Field | Value |
|---|---|
| Change | CR-139 |
| Batch | CR139-W2-DATA-CONTRACTS |
| Stage | CP7 |
| Overall result | PASS_WITH_RISK |
| Test mode | static-only / fixture |
| Runtime/write authorization | Not authorized, not executed |

## Commands

| Command | Result | Summary |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_schema_contract_freeze.py tests/test_cr139_runid_naming.py tests/test_cr139_events_schema_repair.py tests/test_cr139_write_dedup_guarantee.py tests/test_cr139_catalog_manifest_source_of_truth.py tests/test_cr139_lineage_checksum_backfill.py tests/test_cr139_incremental_append_pointer_plan.py tests/test_cr139_runid_lineage_penetration.py tests/test_cr139_cr_data_audit_chain.py tests/test_market_data_normalization_validation_readers.py tests/test_cr139_*.py` | PASS | 91 passed in 4.02s |
| `uv run --python 3.11 pytest -q tests/test_cr014_universe_lifecycle_contract.py tests/test_cr014_p0_pipeline_contract.py tests/test_cr014_windowed_real_run_contract.py tests/test_cr014_readiness_claim_boundary.py tests/test_cr014_incremental_replay_retention.py tests/test_cr014_research_consumer_boundary.py tests/test_cr014_duckdb_readonly_boundary.py tests/test_cr014_unsupported_boundary.py tests/test_cr014_catalog_publish_gate.py tests/test_cr018_publish_current_reader_smoke.py tests/test_cr018_pit_tradability_readiness.py tests/test_cr018_production_current_truth_rerun.py tests/test_cr018_adjustment_publish_readiness.py tests/test_cr018_benchmark_group_readiness.py tests/test_cr018_p1_auxiliary_claim_boundary.py tests/test_cr018_readiness_rollback_gate.py tests/test_cr018_release_scope_dataset_groups.py tests/test_cr134_runner_evidence_index.py tests/test_cr030_experiment_manifest_catalog.py tests/test_cr010_data_lake_publish_and_contracts.py` | PASS | 159 passed in 1.34s |
| `uv run --python 3.11 python -m py_compile engine/contracts.py market_data/readers.py market_data/lake_layout.py market_data/normalization.py market_data/catalog.py market_data/manifest.py market_data/incremental.py market_data/publish.py trading/strategy_runner/evidence_index.py` | PASS | No compile errors |
| `git diff --check` | PASS | No whitespace errors |
| `uv run meta-flow story return-check --packet <story-packet> --return <story-return>` | PASS | 9/9 return packets validated |

The focused W2 command was re-run after review feedback found the earlier manually recorded count was not reproducible. The current recorded count is the actual command output after adding the Gate A no-write guard regressions. This number is a point-in-time snapshot and may grow as guard tests are added; for audit, the command output is the source of truth.

## Test Coverage By Story

| Story | Test File | Coverage |
|---|---|---|
| CR139-S09 | `tests/test_cr139_schema_contract_freeze.py` | schema compatibility, breaking/non-breaking changes, fallback rules, reader fail-closed behavior |
| CR139-S30 | `tests/test_cr139_runid_naming.py` | run-id construction, validation, partition naming contract, legacy path detection |
| CR139-S31 | `tests/test_cr139_events_schema_repair.py` | required events columns, repair metadata, failure on unrecoverable schema gaps |
| CR139-S32 | `tests/test_cr139_write_dedup_guarantee.py` | duplicate key detection, write plan blocking, clean append allowance |
| CR139-S33 | `tests/test_cr139_catalog_manifest_source_of_truth.py` | catalog pointer required fields, manifest/catalog consistency, mismatch failure |
| CR139-S34 | `tests/test_cr139_lineage_checksum_backfill.py` | lineage checksum determinism, manifest derivation, checksum mismatch rejection |
| CR139-S14 | `tests/test_cr139_incremental_append_pointer_plan.py` | append-only plan, pointer advance plan, explicit non-execution guard |
| CR139-S22 | `tests/test_cr139_runid_lineage_penetration.py` | passive evidence lineage consumption, no gateway/trading runtime import |
| CR139-S39 | `tests/test_cr139_cr_data_audit_chain.py` | CR/audit refs in catalog pointer records and manifest pair |
| Gate A guard | `tests/test_cr139_s01_inventory.py`, `tests/test_cr139_s02_golden_baseline.py` | CLI output paths under `lake_root` fail fast before writing |

## Skipped / Not Authorized

| Item | Reason |
|---|---|
| Real lake write | Explicitly not authorized |
| Provider catalog/provider-lake-catalog write | Explicitly not authorized |
| Published pointer advance execution | Explicitly not authorized |
| Physical partition migration | Explicitly not authorized |
| Credential or secret read | Explicitly not authorized |
| NAS/QMT/MiniQMT/gateway runtime | Explicitly not authorized |
| trading/small_live/live | Explicitly not authorized |
| Git remote write | Explicitly not authorized |

## Conclusion

The fixture/static CP7 test suite passes for CR139-W2-DATA-CONTRACTS. The result is `PASS_WITH_RISK` because real lake state has not been mutated or physically verified; those actions require the separate dry-run and execution gates defined in `process/plans/CR139-W2-DATA-LAKE-DRY-RUN-EXECUTION-PLAN.md`.
