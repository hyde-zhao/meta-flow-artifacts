---
story_id: "CR139-S07"
story_slug: "read-layer-dedup"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-28T22:05:41+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S07.CP6.work-packet.json"
---

# CR139-S07 Read-Layer Dedup Implementation

## Implementation Scope

S07 adds read-layer dedup for prices in `market_data/readers.py` and focused fixture tests in `tests/test_cr139_read_layer_dedup.py`.

The implementation introduces `_dataset_dedup_keys()` and `_dedup_by_latest_run()`. `_read_paths()` now attaches a private `_cr139_partition_run_id` column from the `run_id=` path segment, and the dedup helper removes that internal column before returning a public frame. `read_dataset()` and `read_canonical()` both call the helper immediately after parquet concat and before caller filters / column projection.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Dedup by prices key | `_dataset_dedup_keys(DATASET_PRICES)` maps to `("symbol", "trade_date")`; `_dedup_by_latest_run()` drops duplicate key rows | `test_read_dataset_deduplicates_prices_by_catalog_current_run` |
| Catalog current run wins | `_dedup_by_latest_run()` ranks `entry.latest_manifest_run_id` first | `test_read_dataset_deduplicates_prices_by_catalog_current_run` |
| Deterministic fallback when current missing | fallback rank uses `source_run_id` descending and records `catalog_run_id_missing` | `test_read_dataset_uses_deterministic_run_id_fallback_when_catalog_current_missing` |
| Missing `source_run_id` degrades | `_read_paths()` supplies private path run id; helper records `source_run_id_missing` and strips private column | `test_read_dataset_falls_back_to_partition_run_when_source_run_id_missing` |
| Missing dedup keys degrades | helper returns original frame without dedup and records `dedup_keys_missing` | `test_read_dataset_degrades_without_dedup_keys` |
| `read_canonical` sync | `read_canonical()` reuses `_read_paths()` and `_dedup_by_latest_run()` | `test_read_canonical_reuses_read_layer_dedup` |
| No physical migration | implementation only reads tmp_path fixtures and never mutates `run_id=` directories | `test_read_layer_dedup_does_not_migrate_physical_partitions` |

## Changed Files

| File | Change |
|---|---|
| `market_data/readers.py` | Added private partition-run id carrier, dataset dedup key mapping, read-layer dedup helper, and read_dataset/read_canonical integration |
| `tests/test_cr139_read_layer_dedup.py` | Added focused S07 fixture tests |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_read_layer_dedup.py` | PASS, 6 passed in 0.54s |
| `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_panel_reader.py tests/test_cr139_read_layer_dedup.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` | PASS, 62 passed in 2.26s |
| `git diff --check -- market_data/readers.py tests/test_cr139_read_layer_dedup.py` | PASS |

## Boundary Check

| Boundary | Result | Notes |
|---|---|---|
| `market_data/catalog.py` writes | PASS | File not modified; no catalog write introduced by S07 |
| `market_data/manifest.py` / `lake_layout.py` / `normalization.py` / `publish.py` writes | PASS | Not touched |
| Runtime / provider / NAS / QMT | PASS | Not touched |
| Real lake writes | PASS | Tests use tmp_path fixture lake only |
| Published pointer advance execution | PASS | Not executed |
| Physical partition migration | PASS | Not executed; fixture run_id directory hash unchanged |

## Residual Risk

Runtime validation against the real lake remains unauthorized. The fallback ordering before N2 run-id naming governance is deterministic but approximate: catalog current wins first, then `source_run_id` lexicographic descending. S08 remains responsible for partition governance and any physical migration execution remains unauthorized.
