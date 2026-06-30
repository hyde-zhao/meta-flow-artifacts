---
story_id: "CR139-S02"
change_id: "CR-139"
checkpoint: "CP6"
status: "implemented"
implemented_at: "2026-06-28T20:40:58+08:00"
implemented_by: "host-orchestrator-inline"
dispatch_ref: "ADE-CR139-W1-META-DEV-INLINE-S02-START-2026-06-28T204058+0800"
context_ref: "process/context/CP6-CR139-WAVE1-IMPLEMENTATION-CONTEXT.yaml"
---

# CR139-S02 Implementation — T7 Remediation Baseline Golden Values

## Implementation Objects

| Object | Path | Status | Notes |
|---|---|---|---|
| Golden baseline core | `market_data/remediation_baseline.py` | done | Frozen dataclasses, 7 CP5-frozen MetricSpec definitions, deterministic `freeze_golden_baseline`, `compare_golden_baselines`, JSON rendering |
| CLI | `scripts/lake_golden_baseline.py` | done | `freeze` / `compare` subcommands, direct script execution path injection, optional output writing |
| Fixture tests | `tests/test_cr139_s02_golden_baseline.py` | done | S01 inventory contract, all 7 MetricSpec classes, determinism, no lake/catalog mutation, structural and historical attribution |

## Contract Mapping

| Design Contract | Implementation | Verification |
|---|---|---|
| S02 consumes S01 inventory scope | `freeze_golden_baseline()` calls or accepts `build_inventory()` / `InventoryReport` and records `source_inventory_id` | `test_freeze_covers_all_minimum_metric_specs_and_s01_inventory_contract` |
| All 7 CP5-frozen MetricSpec classes are covered | `MINIMUM_METRIC_SPECS` includes `row_count`, `unique_key_dup_count`, `partition_digest`, `pit_reader_summary`, `published_unavailable_count`, `benchmark_window_symbol_set`, `panel_join_row_count` | metric coverage assertion in S02 tests |
| Snapshot output is deterministic | stable frozen_at default, stable JSON sorting, content digests excluding timestamps | `test_freeze_is_bitwise_deterministic_for_same_lake_state` |
| Structural fix and historical changes are attributable | `compare_golden_baselines()` maps structural Story refs and benchmark/window changes to allowed attribution enum | `test_compare_structural_fix_attribution_for_dedup_metric`, `test_compare_historical_window_change_attribution` |
| Only read lake/catalog; write only caller output dir | no provider/NAS/QMT imports; operation counters fixed at zero; freeze writes only `out_dir` when requested | `test_freeze_does_not_modify_catalog_or_lake_files` |
| CLI contract | `scripts/lake_golden_baseline.py freeze --lake-root <root> --out <dir>` and `compare --baseline <dir> --current <dir>` | `test_cli_freeze_and_compare_outputs_json` |

## Validation

Command:

```bash
uv run --python 3.11 pytest -q tests/test_cr139_s02_golden_baseline.py
```

Result: `8 passed in 0.94s`.

Regression command:

```bash
uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py
```

Result: `16 passed in 1.87s`.

## Deviations

No design deviation. To satisfy the CP5 deterministic snapshot requirement, the CLI and library default `frozen_at` / `compared_at` to a stable audit sentinel (`1970-01-01T00:00:00+00:00`) unless the caller explicitly passes a value.

## Remaining Risks

| Risk | Status | Route |
|---|---|---|
| Real lake freeze performance <= 10 minutes | open | Runtime/lake-scale validation remains unauthorized; revisit under scoped runtime authorization |
| Attribution granularity after S04-S08 | controlled | Current rule consumes structural Story refs; refine with follow-up MetricSpec only if ambiguous_rate exceeds 5% in authorized post-structure compare |
