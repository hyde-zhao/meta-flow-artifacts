# CR139 W2 Gate B Batch 2 events Write Execution - 2026-06-29

## Decision

PASS_WITH_RISK_EVENTS_CANDIDATE_WRITE_VERIFIED

`events` was written as an additive-only CR139 split candidate. Metadata-only duplicate primary-key groups were resolved into main with superseded rows quarantined; business-conflict groups were fully quarantined. No catalog/provider catalog/provider-lake-catalog write, published pointer advance, physical partition migration, credential read, NAS/runtime/trading operation, Git remote write, legacy deletion, or rollback deletion was executed.

## Result

| Metric | Value |
|---|---:|
| Source rows | 362,887 |
| Main candidate rows | 350,753 |
| Quarantine rows | 12,134 |
| Metadata-only groups | 10,583 |
| Business-conflict groups | 774 |
| Business-conflict quarantine rows | 1,551 |
| Metadata superseded quarantine rows | 10,583 |
| Main duplicate row over unique | 0 |

## Objects

| Role | Relative path | Rows | Size bytes | SHA-256 |
|---|---|---:|---:|---|
| `main` | `candidate/parquet/dataset=events/schema_version=1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/part-events.parquet` | 350,753 | 5,000,747 | `8ce8517fa53d41f8090ec6c94d5777c2100215ee4d89c2ea282dd389460021b8` |
| `quarantine` | `candidate/parquet/dataset=events/schema_version=1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/quarantine/part-events-conflict-and-superseded.parquet` | 12,134 | 580,578 | `659f6fa9cd52af872d65e4b3d3a82fbadd3d8c1f994ff4fa099b2051044e4f24` |

## Checks

| Check | Result |
|---|---|
| target_absent_before_write | True |
| main_rows_expected | True |
| quarantine_rows_expected | True |
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
| canonical events tree | True |

## Evidence

- Index: `process/evidence/CR139-W2-GATEB-BATCH2-EVENTS-WRITE-EXECUTION.index.json`
- Object manifest: `process/evidence/CR139-W2-GATEB-BATCH2-EVENTS-WRITE-EXECUTION-2026-06-29T210500+0800-OBJECT-MANIFEST.json`
- Rollback manifest: `process/evidence/CR139-W2-GATEB-BATCH2-EVENTS-WRITE-EXECUTION-2026-06-29T210500+0800-ROLLBACK-MANIFEST.json`
- Zero mutation proof: `process/evidence/CR139-W2-GATEB-BATCH2-EVENTS-WRITE-EXECUTION-2026-06-29T210500+0800-ZERO-MUTATION-PROOF.json`

## Next

Continue with `prices` schema-normalized write decision. `prices_limit` and `trade_status` remain single-dataset gates.
