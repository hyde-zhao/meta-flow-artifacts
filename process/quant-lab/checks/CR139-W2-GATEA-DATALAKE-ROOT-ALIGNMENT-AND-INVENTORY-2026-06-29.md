# CR139 W2 Gate A Data-Lake Root Alignment And Inventory

## Scope

| Field | Value |
|---|---|
| Change | CR-139 |
| Batch | CR139-W2-DATA-CONTRACTS |
| Gate | Gate A root alignment and no-write inventory |
| Mode | authorized local lake root rename, then no-write inventory dry-run |
| Previous local root | `/home/hyde/data/quant-lab/lake` |
| Aligned local root | `/home/hyde/data/quant-lab/data-lake` |
| NAS final directory requirement | `/data-lake` |
| Completed at | 2026-06-29T12:28:35+08:00 |
| Decision | PASS_WITH_RISK / READY_FOR_GATE_B_BATCH0_PREFLIGHT_ONLY |

The operator explicitly authorized direct local lake directory rename and stated that NAS already has backup. This check records that root alignment was completed locally. No NAS operation was needed or executed in this step.

No Gate B/C/D/E lake write, catalog write, pointer advance, or physical partition migration was executed.

## Commands

```bash
mv /home/hyde/data/quant-lab/lake /home/hyde/data/quant-lab/data-lake
```

```bash
uv run --python 3.11 python scripts/lake_inventory.py --lake-root /home/hyde/data/quant-lab/data-lake --format json --out process/evidence/CR139-W2-GATEA-REAL-LAKE-INVENTORY-DATALAKE-2026-06-29.json
```

The inventory output path is outside `lake_root`; the Gate A tool guard would fail fast if `--out` resolved inside `lake_root`.

## Entry Criteria

| Criterion | Result | Evidence |
|---|---|---|
| User authorization for local root rename | PASS | User approved direct rename and stated NAS backup exists |
| Target path pre-existence check | PASS | `/home/hyde/data/quant-lab/data-lake` did not exist before rename |
| Local root basename consistency | PASS | New root final segment is `data-lake`, matching NAS `/data-lake` |
| Lake root exists after rename | PASS | `/home/hyde/data/quant-lab/data-lake` |
| Catalog directory exists after rename | PASS | `/home/hyde/data/quant-lab/data-lake/catalog` |
| No credential read required | PASS | Explicit `--lake-root`; `.env` not read |
| Output path outside lake root | PASS | `process/evidence/CR139-W2-GATEA-REAL-LAKE-INVENTORY-DATALAKE-2026-06-29.json` |

## Inventory Summary

| Metric | Old `/lake` Run | New `/data-lake` Run |
|---|---:|---:|
| catalog_dataset_count | 17 | 17 |
| inventory_entry_count | 17 | 17 |
| physical_missing_count | 0 | 0 |
| published_true_count | 0 | 0 |
| published_false_count | 17 | 17 |
| lineage_checksum_present_count | 0 | 0 |
| lineage_checksum_absent_count | 17 | 17 |
| duplicate_key_total | 48,899,668 | 48,899,668 |
| pit_available datasets | 3 | 3 |
| no PIT status datasets | 14 | 14 |

## No-Write Evidence

| Operation Counter | Value |
|---|---:|
| provider_fetch | 0 |
| lake_write | 0 |
| catalog_write | 0 |
| current_pointer_publish | 0 |
| credential_read | 0 |

## Dataset Findings

The root-aligned inventory preserved the same dataset graph:

`adj_factor`, `bse_code_mapping`, `events`, `hs300_index`, `index_members`, `index_weights`, `industry_classification`, `lifecycle_code_change`, `liquidity_capacity`, `market_cap`, `prices`, `prices_limit`, `prices_limit_code_change_fixes`, `prices_limit_coverage_exclusions`, `stock_basic`, `trade_calendar`, `trade_status`.

Classified findings remain unchanged:

| Finding | Severity | Classification | Gate Impact |
|---|---|---|---|
| 17/17 catalog paths use non-CR139 W2 run-id naming. | Medium | Expected CR139 S30 finding | Routes to Gate E planning and/or Gate C path refresh before Gate D pointer advance. |
| 6 datasets have duplicate keys under `(symbol, trade_date)`: `adj_factor`, `liquidity_capacity`, `market_cap`, `prices`, `prices_limit`, `trade_status`. | High | Expected CR139 S32 finding | Blocks writes for affected datasets until Gate B Batch 0 dedup decision review is complete. |
| `events` lacks `trade_date` in inventory columns. | Medium | Expected CR139 S31 finding | Blocks events write execution until repair candidate is reviewed. |
| 17/17 datasets lack lineage checksum. | Medium | Expected CR139 S34 finding | Does not block Gate B Batch 0; blocks Gate C/D readiness until checksum backfill plan is reviewed. |
| 17/17 datasets are unpublished. | High | Current-truth not advanced | Does not block Gate B Batch 0; Gate D remains unauthorized and requires separate pointer review. |
| Physical canonical roots exist for all 17 catalog entries. | Positive | Inventory complete | Supports Gate A completeness criterion. |

## Gate A Pass Threshold

| Criterion | Result | Rationale |
|---|---|---|
| Inventory completeness | PASS | 17 catalog datasets and 17 inventory entries; physical_missing_count=0. |
| Root authenticity | PASS_WITH_RISK | Local root now ends in `data-lake`, matching NAS final directory requirement. Production ownership is operator-confirmed; `.env` was not read. |
| No unexpected external dependency | PASS | Operation counters are zero; command used explicit `--lake-root`; no provider/runtime/credential/NAS/QMT/gateway access was needed. |
| Object graph stability | PASS_WITH_RISK | This session performed no lake data write after rename. No external write fence was mechanically proven; Gate B write batches still require a fenced window or explicit risk acceptance. |
| Finding classification | PASS | Findings map to expected S30/S31/S32/S34/S33/S14/S39 remediation gates; no unknown dataset or physical missing path was found. |

## Decision

Root alignment and Gate A re-inventory are complete.

Next authorized-safe step is Gate B Batch 0 preflight-only:

1. Produce dedup preflight reports for `adj_factor`, `liquidity_capacity`, `market_cap`, `prices`, `prices_limit`, and `trade_status`.
2. Produce `events` repair candidate evidence for missing `trade_date`.
3. Do not write lake objects during Batch 0.
4. Do not proceed to Gate B Batch 1 or Batch 2 writes until the Batch 0 evidence and dataset-specific decisions are reviewed.

## Forbidden Actions Not Executed

- Gate B data object write
- provider catalog/provider-lake-catalog write
- published pointer advance execution
- physical partition migration execution
- credential/secret read
- NAS/QMT/MiniQMT/gateway runtime
- trading/small_live/live
- Git remote write
