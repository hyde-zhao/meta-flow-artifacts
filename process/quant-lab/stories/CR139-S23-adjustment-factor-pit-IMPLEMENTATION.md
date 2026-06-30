---
story_id: "CR139-S23"
story_slug: "adjustment-factor-pit"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-30T11:05:00+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S23.CP6.work-packet.json"
depends_on_standard: "process/stories/CR139-S29-pit-dedup-correctness-tests-LLD.md#8-correctness-standard"
---

# CR139-S23 Adjustment Factor PIT Implementation

## Implementation Scope

S23 adds a pure adjustment-factor PIT selection helper and fixture/static regression tests. The helper selects only factor rows with `available_at <= as_of`, keeps the latest visible row per `(trade_date, symbol)`, and builds a lookup consumed by the existing `_canonical_rows` adjustment path.

The default `normalize_run` write path is not changed. Gate B-verified `adj_factor` write semantics, canonical schemas, real lake data, catalog, manifest, and pointer state remain untouched.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| S29 `STD-S29-PIT-01` / `STD-S29-PIT-02` | `select_pit_adj_factor_frame()` filters by `available_at <= as_of` and selects the latest visible factor | `test_s23_adj_factor_pit_excludes_future_factor_and_applies_visible_factor` |
| REQ-229 timing correctness | `build_pit_adj_factor_lookup()` returns only selected visible factors keyed by `(trade_date, symbol)` | S23 PIT test |
| REQ-229 breakpoint regression | Existing `_canonical_rows` receives the PIT lookup and computes exact adjusted close values across a factor breakpoint | `test_s23_adj_factor_breakpoint_regression_uses_latest_visible_factor` |
| Gate B baseline protection | No change to `_load_adj_factor_lookup()` default call path or `normalize_run()` default write behavior | Targeted regression with S05/S07 tests |

## Changed Files

| File | Change |
|---|---|
| `market_data/normalization.py` | Added `select_pit_adj_factor_frame()` and `build_pit_adj_factor_lookup()` pure helpers; exported them in `__all__`. |
| `tests/test_cr139_adjustment_factor_pit.py` | Added S23 PIT exclusion/application and breakpoint regression tests. |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_pit_correctness.py tests/test_cr139_dedup_correctness.py tests/test_cr139_adjustment_factor_pit.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_read_layer_dedup.py` | PASS, 15 passed in 0.38s |

## Boundary Check

No real lake write, active catalog write, active manifest write, pointer advance, NAS operation, provider fetch, credential read, runtime/trading operation, physical partition migration, or Git remote write was performed.
