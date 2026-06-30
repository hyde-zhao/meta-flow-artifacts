# CR139 Wave1 Verification Report

## S01 T8 Remediation Object Inventory

| Field | Result |
|---|---|
| Story | CR139-S01 |
| Verification mode | static-only / fixture |
| Requirement | REQ-213, REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S01-remediation-object-inventory-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S01.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py` |
| Result | PASS, 8 passed in 1.15s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-213 inventory command | JSON report for all catalog datasets | CLI fixture with 17 datasets |
| REQ-213 duplicate key reporting | `(symbol, trade_date)` duplicate count | known duplicate fixture |
| REQ-247 baseline freeze step 1 | S01 produces S02-consumable inventory | `build_inventory()` dataclass and JSON serialization |
| REQ-248 unauthorized operation boundary | no provider/runtime/catalog write/credential access | catalog hash unchanged and operation counters zero |

### Decision

S01 verification result: `PASS`.

## S02 T7 Remediation Baseline Golden Values

| Field | Result |
|---|---|
| Story | CR139-S02 |
| Verification mode | static-only / fixture |
| Requirement | REQ-212, REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S02-remediation-baseline-golden-values-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S02.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_s02_golden_baseline.py` |
| Result | PASS, 8 passed in 0.94s |
| Regression | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py` -> PASS, 16 passed in 1.87s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-212 golden baseline freeze | GoldenBaselineSnapshot output with source_inventory_id | S01 InventoryReport contract test |
| REQ-212 minimum MetricSpec freeze | All 7 CP5-frozen metric classes present | metric coverage fixture |
| REQ-212 compare attribution | structural_fix / historical_data_change / ambiguous enum | structural dedup and benchmark window change fixtures |
| REQ-247 baseline freeze step 2 | S02 unlocks only after S01 verified | state / story packet update plus S01 CP7 evidence |
| REQ-248 unauthorized operation boundary | no provider/runtime/catalog write/credential access | catalog/tree hash unchanged and operation counters zero |

### Design Contract Verification

| Contract | Result | Evidence |
|---|---|---|
| `freeze_golden_baseline()` consumes S01 `build_inventory()` | PASS | `market_data/remediation_baseline.py`, `test_freeze_covers_all_minimum_metric_specs_and_s01_inventory_contract` |
| `compare_golden_baselines()` produces GoldenDiffReport | PASS | `test_cli_freeze_and_compare_outputs_json` |
| deterministic freeze | PASS | `test_freeze_is_bitwise_deterministic_for_same_lake_state` |
| ambiguous rate controlled for structural fixture | PASS | `test_compare_structural_fix_attribution_for_dedup_metric` |

### Decision

S02 verification result: `PASS`.

## S03 C2a Duplicate Fingerprint Profiling

| Field | Result |
|---|---|
| Story | CR139-S03 |
| Verification mode | static-only / fixture |
| Requirement | REQ-205(C2a), REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S03-duplicate-fingerprint-profiling-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S03.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_duplicate_fingerprint_profiling.py` |
| Result | PASS, 8 passed in 0.67s |
| Regression | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py` -> PASS, 24 passed in 2.14s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-205 C2a duplicate profile | Profile duplicate `(symbol, trade_date)` across canonical `run_id=*` partitions | overlap fixture detects one duplicate key with run ids |
| REQ-205 source lineage | Each duplicate key carries contributing partition run ids and source_run_id cross-check | source_run_id mismatch fixture |
| REQ-247 baseline freeze step 3 | S03 unlocks only after S01/S02 verified and completes baseline gate | state / story packet update plus S01/S02 CP7 evidence |
| REQ-248 unauthorized operation boundary | no provider/runtime/catalog write/credential access/physical migration | tree hash unchanged and operation counters zero |

### Design Contract Verification

| Contract | Result | Evidence |
|---|---|---|
| independent read-only analysis function | PASS | source assertion forbids `read_dataset`, `read_canonical`, `_read_paths`, `_filter_frame` calls in S03 function |
| S01 inventory cross-check | PASS | `test_profile_cross_checks_s01_inventory` |
| 38 partition shape | PASS | `test_profile_38_partition_scale` |
| structured report serialization | PASS | `test_report_is_json_serializable` |

### Decision

S03 verification result: `PASS`. This completes the Wave1 baseline gate (S01/S02/S03 verified) and unlocks CR139-S04 structural implementation. Runtime / real lake profiling remains unauthorized and is recorded as accepted static-only risk.

## S04 V1 Published Pointer / Read Selector

| Field | Result |
|---|---|
| Story | CR139-S04 |
| Verification mode | static-only / fixture |
| Requirement | REQ-232, REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S04-published-pointer-read-selector-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S04.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_published_pointer.py` |
| Result | PASS, 5 passed in 0.49s |
| Regression | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` -> PASS, 46 passed in 2.15s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-232 published current pointer | Catalog selector returns pointer only when entry is published | `test_published_current_pointer_selector_returns_only_published_entry` |
| REQ-232 candidate must remain invisible | Candidate/unpublished entry fails closed | `test_published_current_pointer_selector_blocks_unpublished_candidate` |
| REQ-232 validate pass not publish | validation pass has `publish_count=0`, `current_pointer_changes=0` | `test_validate_pass_does_not_publish_or_advance_current_pointer` |
| REQ-248 no real pointer advance | explicit publish gate dry-run has zero catalog/lake writes | `test_explicit_publish_gate_dry_run_does_not_write_catalog_current_pointer` |
| REQ-232 reader selector | `read_dataset` blocks unpublished and reads published fixture | `test_reader_only_reads_published_catalog_entry` |

### Design Contract Verification

| Contract | Result | Evidence |
|---|---|---|
| published-only selector | PASS | `CatalogStore.get_published_current_pointer()` |
| existing explicit publish gate remains dry-run safe | PASS | CR014 publish gate regression |
| existing P0 read contract preserved | PASS | CR014 P0 pipeline regression |
| no runtime / real lake / pointer advance execution | PASS | fixture tree hashes and zero write assertions |

### Decision

S04 verification result: `PASS`. This unlocks CR139-S05 PIT as-of reader. Real published pointer advance remains not authorized; S05 has an additional hard gate forbidding writes to `market_data/catalog.py`.

## S05 C1 PIT As-Of Reader

| Field | Result |
|---|---|
| Story | CR139-S05 |
| Verification mode | static-only / fixture |
| Requirement | REQ-204, REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S05-pit-as-of-reader-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S05.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_pit_as_of_reader.py` |
| Result | PASS, 5 passed in 0.47s |
| Regression | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr139_pit_as_of_reader.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` -> PASS, 51 passed in 2.39s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-204 PIT as-of read | `available_at <= as_of` filter then latest row per key | `test_read_panel_as_of_filters_future_rows_and_keeps_latest_per_symbol` |
| REQ-204 future financial row leak = 0 | rows whose `available_at` is after `as_of` are absent | same focused fixture |
| REQ-204 missing PIT timestamp fails closed | missing `available_at` returns `required_missing` | `test_read_panel_as_of_requires_available_at` |
| REQ-204 no visible row as of T fails closed | returns empty frame + `pit_as_of_no_available_rows` | `test_read_panel_as_of_returns_required_missing_when_no_rows_available` |
| REQ-232/S04 published gate inheritance | unpublished catalog entry remains `unavailable` and candidate data is not read as current truth | `test_read_panel_as_of_inherits_published_gate_from_read_dataset` |
| REQ-248 catalog.py write forbidden | S05 function does not call `CatalogStore`, `.upsert(`, or `write_text(` | `test_read_panel_as_of_does_not_write_catalog_or_call_catalog_store` |

### Design Contract Verification

| Contract | Result | Evidence |
|---|---|---|
| `read_panel_as_of()` added as single-dataset PIT helper | PASS | `market_data/readers.py` |
| published gate remains first boundary | PASS | S04 regression + S05 unpublished fixture |
| no `market_data/catalog.py` modification | PASS | return boundary check and git touched-file review |
| static-only authorization respected | PASS | tmp_path parquet/catalog fixtures only |

### Decision

S05 verification result: `PASS`. This unlocks CR139-S06 unified panel reader. Runtime validation against real lake data remains unauthorized and is carried as fixture-only risk.

## S06 R1 Unified Panel Reader

| Field | Result |
|---|---|
| Story | CR139-S06 |
| Verification mode | static-only / fixture |
| Requirement | REQ-214, REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S06-unified-panel-reader-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S06.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_panel_reader.py` |
| Result | PASS, 5 passed in 0.26s |
| Regression | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_panel_reader.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` -> PASS, 56 passed in 2.28s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-214 unified panel read | joins prices, financial_pit, market_cap and industry_classification fixture frames with prefixed columns | `test_read_panel_joins_as_of_datasets_with_prefixed_columns` |
| REQ-214 PIT composition | future financial row with `available_at > as_of` absent | same focused fixture |
| REQ-232/S04 published gate consumption | unpublished dataset contributes issue and no prefixed candidate columns | `test_read_panel_records_unpublished_dataset_without_reading_candidate` |
| REQ-214 unknown dataset fails fast | unknown dataset without reader raises `UnknownDatasetError` | `test_read_panel_raises_for_unknown_dataset_without_reader` |
| CR-017 single policy per call | non-string/mixed adjustment policy rejected | `test_read_panel_rejects_mixed_adjustment_policy` |
| REQ-248 catalog.py write forbidden | S06 function composes S05/read_dataset without catalog writes | `test_read_panel_composes_s05_reader_and_does_not_write_catalog` |

### Design Contract Verification

| Contract | Result | Evidence |
|---|---|---|
| `read_panel()` added as unified panel helper | PASS | `market_data/readers.py` |
| S05 PIT helper is composed rather than duplicated | PASS | source check includes `read_panel_as_of` |
| published gate remains first boundary | PASS | source check includes default `read_dataset`; S04/S05 regressions pass |
| no `market_data/catalog.py` modification | PASS | return boundary check and git touched-file review |
| static-only authorization respected | PASS | in-memory fixture reader only |

### Decision

S06 verification result: `PASS`. This unlocks CR139-S07 read-layer dedup. S06 deliberately does not claim dedup correctness; that remains S07 scope.

## S07 C2b Read-Layer Dedup

| Field | Result |
|---|---|
| Story | CR139-S07 |
| Verification mode | static-only / fixture |
| Requirement | REQ-205(C2b), REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S07-read-layer-dedup-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S07.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_read_layer_dedup.py` |
| Result | PASS, 6 passed in 0.54s |
| Regression | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_panel_reader.py tests/test_cr139_read_layer_dedup.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` -> PASS, 62 passed in 2.26s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-205 C2b duplicate key cleanup | `read_dataset(DATASET_PRICES)` returns duplicate key count 0 for `(symbol, trade_date)` | `test_read_dataset_deduplicates_prices_by_catalog_current_run` |
| REQ-205 latest source selection | catalog `latest_manifest_run_id` ranks before older runs | same focused fixture |
| REQ-205 deterministic fallback | missing catalog current falls back to source_run_id lexical descending and records issue | `test_read_dataset_uses_deterministic_run_id_fallback_when_catalog_current_missing` |
| REQ-205 lineage fallback | missing `source_run_id` degrades to path run_id and strips private helper column | `test_read_dataset_falls_back_to_partition_run_when_source_run_id_missing` |
| REQ-205 degrade without keys | missing dedup keys returns original frame with `dedup_keys_missing` | `test_read_dataset_degrades_without_dedup_keys` |
| O-S07-2 read_canonical sync | prices fast reader reuses read-layer dedup | `test_read_canonical_reuses_read_layer_dedup` |
| REQ-248 no physical migration | read-layer dedup does not delete/rename/merge `run_id=` directories | `test_read_layer_dedup_does_not_migrate_physical_partitions` |

### Design Contract Verification

| Contract | Result | Evidence |
|---|---|---|
| `_dedup_by_latest_run()` helper added | PASS | `market_data/readers.py` |
| dedup position is concat after / filter before | PASS | implementation evidence + regression |
| `read_dataset` and `read_canonical` share dedup semantics | PASS | focused fixtures |
| no forbidden file modifications | PASS | return boundary check |
| static-only authorization respected | PASS | tmp_path parquet/catalog fixtures only |

### Decision

S07 verification result: `PASS`. This unlocks CR139-S08 partition governance. Runtime validation against real lake data remains unauthorized. Physical partition migration execution remains explicitly unauthorized and is not part of S07.

## S08 N1 Run ID Partition Key Governance

| Field | Result |
|---|---|
| Story | CR139-S08 |
| Verification mode | static-only / fixture |
| Requirement | REQ-201, REQ-247, REQ-248 |
| Implementation evidence | `process/stories/CR139-S08-runid-partition-key-governance-IMPLEMENTATION.md` |
| CP6 evidence | `process/evidence/STORY-CR139-S08.CP6.index.json` |
| Command | `uv run --python 3.11 pytest -q tests/test_cr139_s08_partition_governance.py` |
| Result | PASS, 10 passed in 0.24s |
| Compatibility regression | `uv run --python 3.11 pytest -q tests/test_market_data_normalization_validation_readers.py` -> PASS, 9 passed in 0.35s |
| Wave1 regression | `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_panel_reader.py tests/test_cr139_read_layer_dedup.py tests/test_cr139_s08_partition_governance.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` -> PASS, 72 passed in 2.26s |

### Traceability

| Scenario / Requirement | Story Contract | Verification |
|---|---|---|
| REQ-201 new canonical path excludes long-lived run_id partition key | `canonical_current_root()` and `_canonical_current_path()` produce `canonical/<dataset>/<schema_version>/current/part-*.parquet` with `run_id=` segment count 0 | `test_current_and_archive_roots_do_not_use_run_id`, `test_normalization_current_path_default_contract_and_legacy_compatibility` |
| REQ-201 current/archive distinction | `LakeLayout` exposes current and archive roots plus partition helpers | `test_current_and_archive_roots_do_not_use_run_id`, `test_current_and_archive_partition_paths_validate_segments` |
| REQ-201 path safety | current/archive partition helpers reject unsafe date, empty, separator, and glob path segments | `test_partition_path_rejects_unsafe_segments` |
| D3 legacy compatibility | old `_canonical_path()` and CLI `run_id=` fallback remain available for existing partitions | `test_normalization_current_path_default_contract_and_legacy_compatibility`, `test_cli_falls_back_to_legacy_run_id_paths_without_migration` |
| N1 dual read transition | CLI path discovery prefers `current/` and falls back to legacy `run_id=` paths | `test_cli_prefers_current_paths_over_legacy_run_id`, `test_cli_falls_back_to_legacy_run_id_paths_without_migration` |
| REQ-201 catalog/manifest metadata contract | `triggered_by_cr` and run_id lineage are metadata, not physical path segments | `test_s08_mock_contract_keeps_run_id_in_metadata_not_path` |
| REQ-248 no physical migration | legacy `run_id=` fixture directories remain present and no real lake operation is executed | `test_cli_falls_back_to_legacy_run_id_paths_without_migration` |

### Design Contract Verification

| Contract | Result | Evidence |
|---|---|---|
| `LakeLayout` current/archive helpers added | PASS | `market_data/lake_layout.py` |
| `normalize_run()` defaults to `partition_layout="current_archive"` | PASS | `market_data/normalization.py` + compatibility regression |
| explicit `legacy_run_id` compatibility retained | PASS | `market_data/normalization.py` + focused fixture |
| `_catalog_canonical_path()` current-first fallback behavior | PASS | `market_data/cli.py` + focused fixture |
| no `catalog.py` / `manifest.py` / `publish.py` modification | PASS | return boundary check |
| physical partition migration not executed | PASS | return boundary check and tmp_path-only fixtures |

### Decision

S08 verification result: `PASS`. CR139 Wave1 S01-S08 are now verified. Physical partition migration, real lake runtime validation, publish supersede movement, catalog `triggered_by_cr`, and manifest lineage persistence remain outside Wave1 execution authorization and must be handled by later active/follow-up stories through context convergence and the required runtime gates.
