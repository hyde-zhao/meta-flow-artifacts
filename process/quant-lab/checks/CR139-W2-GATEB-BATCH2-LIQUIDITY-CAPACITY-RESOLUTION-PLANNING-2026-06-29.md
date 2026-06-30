# CR139 W2 Gate B Batch 2 liquidity_capacity Resolution Planning - 2026-06-29

## Decision

PASS_RESOLUTION_PLAN_READY_FOR_WRITE_AUTHORIZATION_REVIEW

This is a read-only planning gate. It did not write lake objects, catalog records, provider catalog/provider-lake-catalog records, published pointers, physical partitions, NAS, runtime, credentials, Git remote, or rollback actions.

## Resolution Rule

| Field | Value |
|---|---|
| dataset | `liquidity_capacity` |
| duplicate key | `(symbol, trade_date)` |
| conflict type | metadata-only duplicate source lineage |
| selected source_run_id | `run-cr034-chapter3-backfill-2000` |
| superseded source_run_id | `run-cr034-chapter3-smoke-20000104-20000105` |
| precondition | all business columns identical in each duplicate-key group |
| selected-row rationale | full-year backfill run is preferred over smoke-window run when business values are identical |

## Counts

| Metric | Count |
|---|---:|
| source rows | 17095488 |
| non-conflict rows | 17091828 |
| duplicate key groups | 1830 |
| duplicate rows | 3660 |
| selected rows to future main candidate | 1830 |
| superseded rows to future quarantine | 1830 |
| future resolved main row count | 17093658 |
| future resolved quarantine row count | 1830 |
| future total rows preserved | 17095488 |
| dropped rows | 0 |

## Future Candidate Shape

| Object | Relative path | Rows |
|---|---|---:|
| resolved main candidate | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/part-liquidity-capacity.parquet` | 17093658 |
| superseded smoke quarantine | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/quarantine/part-liquidity-capacity-superseded-smoke.parquet` | 1830 |

The current quarantine-only candidate root remains untouched: `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical`.

## Checks

| Check | Result |
|---|---|
| mode is read-only | True |
| source row count matches prior write | True |
| duplicate key group count is 1,830 | True |
| duplicate rows are 3,660 | True |
| all groups have size 2 | True |
| all groups have identical business columns | True |
| selected run present once per group | True |
| superseded run present once per group | True |
| resolved target root absent | True |
| current quarantine candidate left untouched | True |
| operation counts zero | True |

Failed checks: []

## Operation Counts

| Operation | Count |
|---|---:|
| provider_fetch | 0 |
| lake_write | 0 |
| catalog_write | 0 |
| current_pointer_publish | 0 |
| physical_partition_migration | 0 |
| credential_read | 0 |
| nas_operation | 0 |
| runtime_operation | 0 |
| git_remote_write | 0 |

## Evidence

- Planning index: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-RESOLUTION-PLANNING.index.json`
- Planning detail: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-RESOLUTION-PLANNING-2026-06-29T192227+0800.json`

## Next Authorization Needed

A future write still requires separate authorization. Recommended wording:

```text
authorize Gate B Batch 2 liquidity_capacity resolved candidate write only
```

That authorization would write a new additive-only resolved candidate path using `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved`. It would not overwrite the current quarantine-only candidate.
