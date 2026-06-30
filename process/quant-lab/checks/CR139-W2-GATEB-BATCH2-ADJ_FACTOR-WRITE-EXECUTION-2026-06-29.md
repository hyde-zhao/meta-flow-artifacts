# CR139 W2 Gate B Batch 2 adj_factor Write Execution - 2026-06-29

## Decision

PASS_WITH_RISK_ADJ_FACTOR_CANDIDATE_WRITE_VERIFIED

`adj_factor` was written as an additive-only CR139 candidate. No catalog/provider catalog/provider-lake-catalog write, published pointer advance, physical partition migration, credential read, NAS/runtime/trading operation, Git remote write, legacy deletion, or rollback deletion was executed.

## Scope

Authorized scope: authorize Gate B Batch 2 adj_factor exact-dedup + metadata-resolved additive-only candidate write only

## Result

| Metric | Value |
|---|---:|
| Source rows | 29,970,830 |
| Main candidate rows | 17,905,960 |
| Quarantine rows | 321,815 |
| Dropped exact duplicate rows | 11,743,055 |
| Accounted rows | 29,970,830 |
| Main duplicate row over unique | 0 |

## Objects

| Role | Relative path | Rows | Size bytes | SHA-256 |
|---|---|---:|---:|---|
| `main` | `candidate/parquet/dataset=adj_factor/schema_version=1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/part-adj-factor.parquet` | 17,905,960 | 77,729,543 | `8fff4932d608d268bb6d0766caa7797be7ec9f74d64266f9b55e08a5ed0a5428` |
| `metadata_superseded_quarantine` | `candidate/parquet/dataset=adj_factor/schema_version=1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/quarantine/part-adj-factor-superseded-metadata.parquet` | 321,815 | 784,970 | `fad437c3b6bdf8f39cebfe91b91a1178e8002a339d753a41e6f1d7b7cbeb236f` |

## Checks

| Check | Result |
|---|---|
| target_absent_before_write | True |
| main_rows_expected | True |
| quarantine_rows_expected | True |
| dropped_exact_duplicates_expected | True |
| main_has_no_duplicate_keys | True |
| zero_mutation_pass | True |
| rollback_not_authorized | True |

## Zero Mutation

| Protected surface | Unchanged |
|---|---|
| catalog tree | True |
| published tree | True |
| canonical adj_factor tree | True |

## Evidence

- Index: `process/evidence/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION.index.json`
- Object manifest: `process/evidence/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION-2026-06-29T203800+0800-OBJECT-MANIFEST.json`
- Rollback manifest: `process/evidence/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION-2026-06-29T203800+0800-ROLLBACK-MANIFEST.json`
- Zero mutation proof: `process/evidence/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION-2026-06-29T203800+0800-ZERO-MUTATION-PROOF.json`

## Next

Continue with `events` decision planning. `prices`, `prices_limit`, and `trade_status` remain decision-gated and must not be batch-written without per-dataset review.
