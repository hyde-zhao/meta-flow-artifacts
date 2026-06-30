# CR139 W2 Gate B Batch 2 trade_status Write Execution - 2026-06-29

## Decision

PASS_WITH_RISK_TRADE_STATUS_CANDIDATE_WRITE_VERIFIED

`trade_status` was written as an additive-only CR139 split candidate. Exact-copy duplicate rows were deduplicated, metadata-only duplicate-key groups were resolved into main with superseded rows quarantined, and business-conflict groups were fully quarantined. No catalog/provider catalog/provider-lake-catalog write, published pointer advance, physical partition migration, credential read, NAS/runtime/trading operation, Git remote write, legacy deletion, or rollback deletion was executed.

## Result

| Metric | Value |
|---|---:|
| Source rows | 21,569,907 |
| Main candidate rows | 13,957,403 |
| Quarantine rows | 7,612,504 |
| Dropped exact duplicate rows | 0 |
| Metadata-only groups | 70,087 |
| Business-conflict groups | 3,692,105 |
| Business-conflict quarantine rows | 7,542,079 |
| Metadata superseded quarantine rows | 70,425 |
| Main duplicate row over unique | 0 |
| Accounted row count | 21,569,907 |

## Objects

| Role | Relative path | Rows | Size bytes | SHA-256 |
|---|---|---:|---:|---|
| `main` | `candidate/parquet/dataset=trade_status/schema_version=1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/part-trade-status.parquet` | 13,957,403 | 35,490,160 | `19de5b332398e0f89358bed887ae3df215508c3b6a7d73407d6144c12f03789f` |
| `quarantine` | `candidate/parquet/dataset=trade_status/schema_version=1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/quarantine/part-trade-status-conflict-and-superseded.parquet` | 7,612,504 | 17,659,218 | `42e5bc36f1fecbaa33db4d8696b748ed29f4b75bde867dfed978e7c14870ce16` |

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

- Index: `process/evidence/CR139-W2-GATEB-BATCH2-TRADE_STATUS-WRITE-EXECUTION.index.json`
- Object manifest: `process/evidence/CR139-W2-GATEB-BATCH2-TRADE_STATUS-WRITE-EXECUTION-2026-06-29T214000+0800-OBJECT-MANIFEST.json`
- Rollback manifest: `process/evidence/CR139-W2-GATEB-BATCH2-TRADE_STATUS-WRITE-EXECUTION-2026-06-29T214000+0800-ROLLBACK-MANIFEST.json`
- Zero mutation proof: `process/evidence/CR139-W2-GATEB-BATCH2-TRADE_STATUS-WRITE-EXECUTION-2026-06-29T214000+0800-ZERO-MUTATION-PROOF.json`
