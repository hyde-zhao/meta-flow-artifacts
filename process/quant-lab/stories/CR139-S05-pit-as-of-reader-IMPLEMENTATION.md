---
story_id: "CR139-S05"
story_slug: "pit-as-of-reader"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-28T21:39:21+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S05.CP6.work-packet.json"
---

# CR139-S05 PIT As-Of Reader Implementation

## Implementation Scope

S05 adds `read_panel_as_of()` in `market_data/readers.py` and fixture tests in `tests/test_cr139_pit_as_of_reader.py`.

The function is a single-dataset PIT read helper. By default it calls `read_dataset()`, so the published catalog gate remains the first boundary. After that gate passes, it requires key columns plus `available_at`, filters rows with `available_at <= as_of`, sorts by keys plus `trade_date` when present and `available_at`, then keeps the latest visible row per key.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Preserve published current truth boundary | `read_panel_as_of()` calls `read_dataset()` by default and normalizes `catalog_not_published` to `unavailable` | `test_read_panel_as_of_inherits_published_gate_from_read_dataset` |
| No future row leakage | filters `available_at <= as_of` before deduplication | `test_read_panel_as_of_filters_future_rows_and_keeps_latest_per_symbol` |
| Missing `available_at` fails closed | returns `required_missing` with `pit_as_of_required_column_missing` | `test_read_panel_as_of_requires_available_at` |
| No visible row as of T fails closed | returns empty frame plus `pit_as_of_no_available_rows` | `test_read_panel_as_of_returns_required_missing_when_no_rows_available` |
| No catalog.py write in S05 | implementation touches only `readers.py`; test statically checks no `CatalogStore`, `.upsert(`, or `write_text(` inside S05 function | `test_read_panel_as_of_does_not_write_catalog_or_call_catalog_store` |

## Changed Files

| File | Change |
|---|---|
| `market_data/readers.py` | Added `read_panel_as_of()` and exported it via `__all__`; added published-gate status normalization for `catalog_not_published` |
| `tests/test_cr139_pit_as_of_reader.py` | Added focused PIT as-of reader fixture tests |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_pit_as_of_reader.py` | PASS, 5 passed in 0.47s |
| `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr139_pit_as_of_reader.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py` | PASS, 51 passed in 2.39s |

An initial S05 focused run failed because `read_dataset(required=True)` maps unpublished catalog entries to `required_missing`. The implementation was corrected so `catalog_not_published` remains `unavailable` at the S05 boundary, preserving the S04 published-only contract.

## Boundary Check

| Boundary | Result | Notes |
|---|---|---|
| `market_data/catalog.py` writes | PASS | No modification or write call in S05 implementation |
| Runtime / provider / NAS / QMT | PASS | Not touched |
| Real lake writes | PASS | Tests write only tmp_path fixture parquet/catalog |
| Published pointer advance execution | PASS | Not executed |
| Physical partition migration | PASS | Not executed |

## Residual Risk

Runtime validation against a real lake remains unauthorized. The current evidence is fixture-only and static, which is the approved Wave1 validation mode.
