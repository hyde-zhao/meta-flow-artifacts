---
story_id: "CR139-S08"
story_slug: "runid-partition-key-governance"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-28T22:21:23+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S08.CP6.work-packet.json"
---

# CR139-S08 Run ID Partition Key Governance Implementation

## Implementation Scope

S08 implements the path-governance slice of N1. New canonical output now has a current/archive path contract through `market_data/lake_layout.py`, `normalize_run()` defaults new writes to `canonical/<dataset>/<schema_version>/current/part-*.parquet`, and CLI canonical path discovery prefers `current/` while preserving legacy `run_id=` fallback.

The implementation deliberately does not execute physical partition migration. It also does not implement publish supersede movement, catalog `triggered_by_cr` persistence, or manifest lineage persistence in `publish.py`, `catalog.py`, or `manifest.py`; those remain cross-boundary consumer contracts for later active stories.

One comment-only change in `market_data/readers.py` removes a forbidden import-boundary test token from a defensive-read comment. It does not change reader behavior.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| canonical current/archive roots | `LakeLayout.canonical_current_root()` and `canonical_archive_root()` return `canonical/<dataset>/<schema_version>/current|archive` without directory creation | `test_current_and_archive_roots_do_not_use_run_id` |
| current/archive partition helpers | `canonical_current_partition_path()` and `canonical_archive_partition_path()` reuse `_compact_date` and `_clean_partition_value` | `test_current_and_archive_partition_paths_validate_segments`, `test_partition_path_rejects_unsafe_segments` |
| new data excludes `run_id=` path segment | `_canonical_current_path()` targets `current/part-*.parquet`; `normalize_run()` defaults `partition_layout="current_archive"` | `test_normalization_current_path_default_contract_and_legacy_compatibility`, existing normalization regression |
| legacy compatibility remains explicit | `_canonical_path()` is preserved and `normalize_run(partition_layout="legacy_run_id")` remains available | `test_normalization_current_path_default_contract_and_legacy_compatibility` |
| CLI current-first dual read | `_canonical_paths_for_run()` and `_catalog_canonical_path()` prefer `current/`, then fallback to legacy `run_id=` paths | `test_cli_prefers_current_paths_over_legacy_run_id`, `test_cli_falls_back_to_legacy_run_id_paths_without_migration` |
| cross-boundary metadata contract | Mock fixture keeps `triggered_by_cr` in catalog metadata and `run_id` in manifest metadata, not in physical path | `test_s08_mock_contract_keeps_run_id_in_metadata_not_path` |
| no physical migration | Implementation never deletes, renames, merges, or moves legacy `run_id=` fixture directories | `test_cli_falls_back_to_legacy_run_id_paths_without_migration` |

## Changed Files

| File | Change |
|---|---|
| `market_data/lake_layout.py` | Added canonical current/archive root and partition path helpers |
| `market_data/normalization.py` | Added partition layout constants, `_canonical_current_path()`, and default current/archive target selection in `normalize_run()` |
| `market_data/cli.py` | Added current-first canonical path discovery and catalog path selection with legacy fallback |
| `market_data/readers.py` | Comment-only guard-test wording fix from S07 regression |
| `tests/test_cr139_s08_partition_governance.py` | Added focused S08 fixture tests |
| `tests/test_market_data_normalization_validation_readers.py` | Updated default normalize path expectation to `current/` |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_s08_partition_governance.py` | PASS, 10 passed in 0.24s |
| `uv run --python 3.11 pytest -q tests/test_market_data_normalization_validation_readers.py` | PASS, 9 passed in 0.35s |
| `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_panel_reader.py tests/test_cr139_read_layer_dedup.py tests/test_cr139_s08_partition_governance.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` | PASS, 72 passed in 2.26s |
| `git diff --check -- market_data/lake_layout.py market_data/normalization.py market_data/cli.py market_data/readers.py tests/test_cr139_s08_partition_governance.py tests/test_market_data_normalization_validation_readers.py` | PASS |

## Boundary Check

| Boundary | Result | Notes |
|---|---|---|
| `market_data/catalog.py` writes | PASS | File not modified; catalog `triggered_by_cr` remains mock contract only |
| `market_data/manifest.py` writes | PASS | File not modified; run_id lineage persistence remains consumer contract only |
| `market_data/publish.py` writes | PASS | File not modified; current/archive supersede movement remains consumer contract only |
| Runtime / provider / NAS / QMT | PASS | Not touched |
| Real lake writes | PASS | Tests use tmp_path fixtures only |
| Published pointer advance execution | PASS | Not executed |
| Physical partition migration | PASS | Not executed; legacy `run_id=` fallback fixture remains in place |

## Residual Risk

Physical partition migration is still not authorized and has not run. Supersede movement from `current/` to `archive/`, catalog `triggered_by_cr`, manifest run-id lineage persistence, and run-lineage catalog metadata remain downstream consumer-story obligations. S08 only proves the path-governance contract and fixture-level compatibility.
