# CR139 W2 Gate B Batch 2 liquidity_capacity Write Execution Check - 2026-06-29

## Decision

PASS_WITH_RISK - `liquidity_capacity` additive quarantine-only candidate write completed and verified.

## Authorized Scope

- Authorized dataset: `liquidity_capacity` only.
- Authorized write mode: additive-only new CR139 candidate path with quarantine subpath.
- Not authorized: catalog writes, provider catalog/provider-lake-catalog writes, published pointer advance, physical migration, credential reads, NAS operation, runtime/QMT/gateway/trading, Git remote write, rollback deletion, and all other datasets.

## Written Objects

| Role | Relative path | Rows | SHA-256 |
|---|---|---:|---|
| candidate main non-conflict rows | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical/part-liquidity-capacity.parquet` | 17091828 | `07e36c7987713ad3424dd5942c90817b95e8c702f58748c5251f139d346a5a21` |
| quarantine metadata-conflict key groups | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical/quarantine/part-liquidity-capacity-conflict-groups.parquet` | 3660 | `c1680cc06e8e8968418c4ead707f7aa6cffde2deaff16206aea614a4110822ce` |

## Verification

| Check | Result |
|---|---|
| target absent before write | True |
| source row count matches profile: 17,095,488 | True |
| duplicate key groups match profile: 1,830 | True |
| duplicate rows in conflict groups match profile: 3,660 | True |
| candidate main row count: 17,091,828 | True |
| quarantine row count: 3,660 | True |
| combined target row count equals source | True |
| candidate main duplicate-key over-unique count is 0 | True |
| quarantine contains all duplicate-key groups | True |
| source vs constructed target row-hash multiset match | True |
| source vs readback target row-hash multiset match | True |
| catalog hash unchanged | True |
| published tree hash unchanged | True |
| canonical liquidity_capacity tree hash unchanged | True |
| operation counts exact | True |
| rollback not authorized | True |

Failed checks: []

## Operation Counts

| Operation | Count |
|---|---:|
| provider_fetch | 0 |
| lake_write | 2 |
| catalog_write | 0 |
| current_pointer_publish | 0 |
| physical_partition_migration | 0 |
| credential_read | 0 |
| nas_operation | 0 |
| runtime_operation | 0 |
| git_remote_write | 0 |

## Evidence

- Pre-write snapshot: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION-2026-06-29T184303+0800-PREWRITE-SNAPSHOT.json`
- Post-write snapshot: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION-2026-06-29T184303+0800-POSTWRITE-SNAPSHOT.json`
- Object manifest: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION-2026-06-29T184303+0800-OBJECT-MANIFEST.json`
- Rollback manifest: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION-2026-06-29T184303+0800-ROLLBACK-MANIFEST.json`
- Zero pointer mutation proof: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION-2026-06-29T184303+0800-ZERO-POINTER-MUTATION.json`
- Execution index: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION.index.json`

## Residual Risk

The conflict groups are quarantined, not resolved. Gate B did not select a published truth row for the 1,830 duplicate keys. Any future catalog/pointer/publish step must review this quarantine evidence and decide whether to keep quarantine, repair source lineage, or select a canonical row under a separate authorization.
