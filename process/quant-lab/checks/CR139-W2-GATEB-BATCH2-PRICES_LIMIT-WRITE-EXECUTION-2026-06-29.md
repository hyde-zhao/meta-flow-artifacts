# CR139 W2 Gate B Batch 2 prices_limit Write Execution - 2026-06-29

## Decision

PASS_WITH_RISK_PRICES_LIMIT_CANDIDATE_WRITE_VERIFIED

`prices_limit` was written as an additive-only CR139 split candidate. Exact-copy duplicate rows were deduplicated, metadata-only duplicate-key groups were resolved into main with superseded rows quarantined, and business-conflict groups were fully quarantined. No catalog/provider catalog/provider-lake-catalog write, published pointer advance, physical partition migration, credential read, NAS/runtime/trading operation, Git remote write, legacy deletion, or rollback deletion was executed.

## Result

| Metric | Value |
|---|---:|
| Source rows | 40,962,525 |
| Main candidate rows | 19,210,017 |
| Quarantine rows | 11,841,929 |
| Dropped exact duplicate rows | 9,910,579 |
| Metadata-only groups | 6,946,008 |
| Business-conflict groups | 499,787 |
| Business-conflict quarantine rows | 1,261,096 |
| Metadata superseded quarantine rows | 10,580,833 |
| Main duplicate row over unique | 0 |
| Accounted row count | 40,962,525 |

## Objects

| Role | Relative path | Rows | Size bytes | SHA-256 |
|---|---|---:|---:|---|
| `main` | `candidate/parquet/dataset=prices_limit/schema_version=1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/part-prices-limit.parquet` | 19,210,017 | 117,729,976 | `0f85e1583e77e3b9769f2f7ae5622e7086955fe7437ac33e1fce885e77bebc72` |
| `quarantine` | `candidate/parquet/dataset=prices_limit/schema_version=1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/quarantine/part-prices-limit-conflict-and-superseded.parquet` | 11,841,929 | 67,212,938 | `5d787eae6c90a90ec9c835b2378fcd06de2efbf9cfbc929944a5a5dced504c05` |

## Checks

| Check | Result |
|---|---|
| target_absent_before_write | True |
| source_rows_expected | True |
| main_rows_expected | True |
| quarantine_rows_expected | True |
| dropped_exact_duplicates_expected | True |
| metadata_only_groups_expected | True |
| business_conflict_groups_expected | True |
| main_has_no_duplicate_keys | True |
| zero_mutation_pass | True |
| rollback_not_authorized | True |

## Zero Mutation

| Protected surface | Unchanged |
|---|---|
| catalog tree | True |
| published tree | True |
| canonical dataset tree | True |

## Evidence

- Index: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES_LIMIT-WRITE-EXECUTION.index.json`
- Object manifest: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES_LIMIT-WRITE-EXECUTION-2026-06-29T213000+0800-OBJECT-MANIFEST.json`
- Rollback manifest: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES_LIMIT-WRITE-EXECUTION-2026-06-29T213000+0800-ROLLBACK-MANIFEST.json`
- Zero mutation proof: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES_LIMIT-WRITE-EXECUTION-2026-06-29T213000+0800-ZERO-MUTATION-PROOF.json`
