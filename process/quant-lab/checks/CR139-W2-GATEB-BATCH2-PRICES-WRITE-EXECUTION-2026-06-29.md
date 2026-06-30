# CR139 W2 Gate B Batch 2 prices Write Execution - 2026-06-29

## Decision

PASS_WITH_RISK_PRICES_CANDIDATE_WRITE_VERIFIED

`prices` was written as an additive-only CR139 split candidate. Exact-copy duplicate rows were deduplicated, metadata-only duplicate-key groups were resolved into main with superseded rows quarantined, and business-conflict groups were fully quarantined. No catalog/provider catalog/provider-lake-catalog write, published pointer advance, physical partition migration, credential read, NAS/runtime/trading operation, Git remote write, legacy deletion, or rollback deletion was executed.

## Result

| Metric | Value |
|---|---:|
| Source rows | 28,646,730 |
| Main candidate rows | 17,013,700 |
| Quarantine rows | 401,628 |
| Dropped exact duplicate rows | 11,231,402 |
| Metadata-only groups | 1,830 |
| Business-conflict groups | 79,958 |
| Business-conflict quarantine rows | 399,798 |
| Metadata superseded quarantine rows | 1,830 |
| Main duplicate row over unique | 0 |
| Accounted row count | 28,646,730 |

## Objects

| Role | Relative path | Rows | Size bytes | SHA-256 |
|---|---|---:|---:|---|
| `main` | `candidate/parquet/dataset=prices/schema_version=1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/part-prices.parquet` | 17,013,700 | 778,036,005 | `1bf74d76705470923629dcec0242f681cc4f7724a2a63496cbe167e4830d7093` |
| `quarantine` | `candidate/parquet/dataset=prices/schema_version=1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/quarantine/part-prices-conflict-and-superseded.parquet` | 401,628 | 9,307,225 | `c074722d4c383189cf2b1a6eb11b48ec5e660090d753d56676c8d6acd4de02d0` |

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

- Index: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES-WRITE-EXECUTION.index.json`
- Object manifest: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES-WRITE-EXECUTION-2026-06-29T212000+0800-OBJECT-MANIFEST.json`
- Rollback manifest: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES-WRITE-EXECUTION-2026-06-29T212000+0800-ROLLBACK-MANIFEST.json`
- Zero mutation proof: `process/evidence/CR139-W2-GATEB-BATCH2-PRICES-WRITE-EXECUTION-2026-06-29T212000+0800-ZERO-MUTATION-PROOF.json`
