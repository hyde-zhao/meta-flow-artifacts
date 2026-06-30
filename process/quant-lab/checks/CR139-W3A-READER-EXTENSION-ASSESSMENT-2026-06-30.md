# CR139 W3-A Reader Extension Assessment

- status: `pass_w3a_reader_extension_assessment`
- evidence: `/home/hyde/workspace/quant-lab/process/evidence/CR139-W3A-READER-EXTENSION-ASSESSMENT-2026-06-30.json`
- read_dataset_location: `market_data/readers.py:3037`

## P0 Datasets

| Dataset | In DATASETS | Has Schema | Current read_dataset Status | Rows | Row Count Matches |
|---|---:|---:|---|---:|---:|
| `liquidity_capacity` | False | False | `unavailable` | 17093658 | True |
| `market_cap` | False | False | `unavailable` | 17001327 | True |

## Checks

| Check | Result |
|---|---|
| `active_catalog_has_p0_datasets` | PASS |
| `p0_canonical_paths_exist` | PASS |
| `p0_row_counts_match_catalog` | PASS |
| `p0_reader_currently_unknown` | PASS |
| `extension_scope_identified` | PASS |
| `no_lake_write` | PASS |

## Recommendation

- Implement P0 reader support by adding `liquidity_capacity` and `market_cap` constants to `market_data.contracts.DATASETS` plus schema registry entries derived from CR139 canonical parquet columns.
- Reuse the generic `read_dataset()` path; add tests that current read status changes from `unknown_dataset` to `available` and row counts match active catalog.
- Avoid full-table performance assertions; use metadata row counts and one actual read smoke per dataset, with optional column/sample smoke if runtime grows.
