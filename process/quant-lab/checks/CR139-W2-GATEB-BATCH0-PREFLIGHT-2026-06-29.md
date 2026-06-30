# CR139 W2 Gate B Batch 0 Preflight

## Scope

| Field | Value |
|---|---|
| Change | CR-139 |
| Batch | CR139-W2-DATA-CONTRACTS |
| Gate | Gate B |
| Batch step | Batch 0 preflight-only |
| Lake root | `/home/hyde/data/quant-lab/data-lake` |
| Evidence | `process/evidence/CR139-W2-GATEB-BATCH0-PREFLIGHT-2026-06-29.json` |
| Decision | BLOCKS_GATE_B_WRITE_BATCHES / DECISION_REQUIRED |

Batch 0 was executed as read-only preflight evidence generation. It did not write lake objects, catalog/provider-lake-catalog records, published pointers, or physical migrations.

## Commands

```bash
uv run --python 3.11 python scripts/cr139_gateb_batch0_preflight.py --lake-root /home/hyde/data/quant-lab/data-lake --out process/evidence/CR139-W2-GATEB-BATCH0-PREFLIGHT-2026-06-29.json --sample-limit 20
```

## Tool Guard

| Guard | Result | Evidence |
|---|---|---|
| Output outside lake root | PASS | `--out` path resolves under `process/evidence`, not `/home/hyde/data/quant-lab/data-lake` |
| Provider/runtime/NAS/credential access | PASS | Script imports only local parquet/catalog/contract modules |
| Lake write/catalog write/pointer advance | PASS | `operation_counts` all zero |
| Test coverage | PASS | `uv run --python 3.11 pytest -q tests/test_cr139_gateb_batch0_preflight.py` -> 2 passed |

## No-Write Evidence

| Operation Counter | Value |
|---|---:|
| provider_fetch | 0 |
| lake_write | 0 |
| catalog_write | 0 |
| current_pointer_publish | 0 |
| credential_read | 0 |

## Dedup Preflight Results

| Dataset | Status | Rows | Unique Keys | Duplicate Rows Over Unique | Decision Required |
|---|---|---:|---:|---:|---|
| adj_factor | requires_decision | 29,970,830 | 17,905,960 | 12,064,870 | yes |
| liquidity_capacity | requires_decision | 17,095,488 | 17,093,658 | 1,830 | yes |
| market_cap | requires_decision | 17,108,103 | 17,001,327 | 106,776 | yes |
| prices | requires_decision | 28,646,730 | 17,093,658 | 11,553,072 | yes |
| prices_limit | requires_decision | 40,962,525 | 19,709,804 | 21,252,721 | yes |
| trade_status | requires_decision | 21,569,907 | 17,649,508 | 3,920,399 | yes |

All six duplicate-key datasets require a dataset-specific keep/drop/quarantine decision before any affected dataset write.

## Events Repair Preflight

| Field | Value |
|---|---|
| status | blocked |
| rows | 362,887 |
| canonical missing columns | none |
| extra columns | none |
| trade_date column present | false |
| trade_date required by current W2 events contract | false |
| blocking error | duplicate_key |

Important clarification: the current W2 `CANONICAL_EVENTS_COLUMNS` contract does not include `trade_date`; events repair is blocked because `repair_events_schema()` fails closed on duplicate primary keys under `(symbol, event_type, event_date, available_at)`, not because `trade_date` is missing from the current contract.

## Decision

Gate B Batch 0 completed, but it blocks Gate B write batches until decisions are made.

Required before Batch 1/2 writes:

1. For each duplicate-key dataset, approve a deterministic keep/drop/quarantine rule.
2. For `events`, decide whether to deduplicate under `(symbol, event_type, event_date, available_at)`, quarantine duplicate events, or revise the events primary-key contract through a design delta before writing.
3. Keep Batch 1 limited to zero-duplicate datasets only if write authorization is still desired after Batch 0 review.
4. Do not execute catalog writes, pointer advance, or physical migration under this decision.

## Forbidden Actions Not Executed

- Gate B data object write
- provider catalog/provider-lake-catalog write
- published pointer advance execution
- physical partition migration execution
- credential/secret read
- NAS/QMT/MiniQMT/gateway runtime
- trading/small_live/live
- Git remote write
