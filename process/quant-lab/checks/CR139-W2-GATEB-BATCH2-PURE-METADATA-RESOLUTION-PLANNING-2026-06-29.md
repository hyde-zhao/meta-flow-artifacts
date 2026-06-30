# CR139 W2 Gate B Batch 2 Pure Metadata Resolution Planning - 2026-06-29

## Decision

BLOCKED_STRATEGY_UNIFICATION_DECISIONS_REQUIRED_BEFORE_BATCH_WRITE

The review finding is accepted. `market_cap` is metadata-only and the aux precedence recommendation is reasonable, but combined batch write is blocked because `liquidity_capacity` already has a quarantine-only candidate while the newer plan recommends a resolved strategy.

## Blocking Finding

| Dataset | Existing candidate | Existing strategy | Resolved strategy | Impact |
|---|---|---|---|---|
| `liquidity_capacity` | `cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical` | main 17,091,828 + quarantine 3,660 | `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` main 17,093,658 + quarantine 1,830 | Existing main is missing 1,830 keys; writing market_cap resolved while leaving liquidity unresolved creates inconsistent candidate semantics. |

## Required Decisions

| Decision ID | Recommended | Alternative | Impact |
|---|---|---|---|
| D-GATEB-B2-LIQUIDITY-001 | Roll back exactly two old liquidity candidate objects and write resolved liquidity candidate | Keep old quarantine-only candidate | Recommended path unifies strategy and keeps liquidity main complete; requires explicit rollback/delete authorization. |
| D-GATEB-B2-MARKETCAP-001 | Select `run-stage3-data-update-20260530-20260626-aux-20260627` and quarantine `run-stage3-data-update-20260530-20260626-prices-20260627` | Select prices run and quarantine aux run | Business values are identical; decision affects lineage ownership and audit interpretation. |

## Future Execution Plan After Approval

| Batch | Datasets | Mode | Status | Expected operations |
|---|---|---|---|---|
| `B2-PURE-METADATA-UNIFICATION-EXECUTION` | `liquidity_capacity, market_cap` | compound candidate cleanup plus resolved metadata-only candidate writes | blocked_until_D-GATEB-B2-LIQUIDITY-001_and_D-GATEB-B2-MARKETCAP-001_approved | {'candidate_object_delete': 2, 'lake_write': 4, 'catalog_write': 0, 'current_pointer_publish': 0, 'physical_partition_migration': 0} |
| `B2-PURE-METADATA-LIQUIDITY-ONLY-RESOLUTION` | `liquidity_capacity` | rollback old quarantine-only candidate then write resolved candidate | available_if_market_cap_decision_is_deferred_but_D-GATEB-B2-LIQUIDITY-001_approved | {'candidate_object_delete': 2, 'lake_write': 2, 'catalog_write': 0, 'current_pointer_publish': 0, 'physical_partition_migration': 0} |
| `B2-MOSTLY-EXACT-PLANNING` | `adj_factor, prices` | read-only split exact-dedup plus conflict quarantine planning before any write | planning_needed_can_run_in_parallel_with_pure_metadata_execution_review | {} |
| `B2-PRIMARY-OR-MOSTLY-CONFLICT-SINGLETONS` | `events, trade_status` | single-dataset planning/write gates; not batch write until field semantics are reviewed | planning_needed_can_run_in_parallel | {} |
| `B2-MIXED-PRICES-LIMIT` | `prices_limit` | single-dataset split exact-dedup plus quarantine planning/write due mixed conflict scale | planning_needed_can_run_in_parallel | {} |


## Operation Counts For This Planning Revision

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

- Planning index: `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-RESOLUTION-PLANNING.index.json`
- Planning detail: `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-RESOLUTION-PLANNING-2026-06-29T193115+0800.json`
- Prior liquidity write index: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION.index.json`

## Recommendation

Approve both decisions, then authorize a single compound Gate B execution for pure metadata strategy unification:

```text
approve D-GATEB-B2-LIQUIDITY-001 option A and D-GATEB-B2-MARKETCAP-001; authorize Gate B Batch 2 pure metadata strategy-unification execution only for liquidity_capacity and market_cap
```

That future authorization must explicitly allow deleting only the two old `liquidity_capacity` candidate objects from the prior manifest and writing four new resolved candidate objects. It must still forbid catalog, pointer, physical migration, NAS, runtime, credential, Git remote, and any legacy deletion.


## Review Feedback Adoption - Delete Guard

The review feedback is accepted and adopted. `D-GATEB-B2-MARKETCAP-001` is recorded as approved: select aux run and quarantine prices run. `D-GATEB-B2-LIQUIDITY-001` option A is recorded as approved only with the mandatory delete guard below.

### GATEB-B2-LIQUIDITY-DELETE-GUARD-001

Allowed delete mode: exact prior manifest object list only.

Allowed object paths:

| Relative path | Expected rows | Expected size | Expected SHA-256 |
|---|---:|---:|---|
| `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical/part-liquidity-capacity.parquet` | 17091828 | 650227690 | `07e36c7987713ad3424dd5942c90817b95e8c702f58748c5251f139d346a5a21` |
| `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical/quarantine/part-liquidity-capacity-conflict-groups.parquet` | 3660 | 136155 | `c1680cc06e8e8968418c4ead707f7aa6cffde2deaff16206aea614a4110822ce` |


Pre-delete checks required:

- path is relative and under `candidate/`;
- path contains exact old run id `cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical`;
- path has no wildcard/glob characters;
- path is not under `canonical/`, `published/`, `catalog/`, or any legacy source tree;
- file exists and sha256/size/row_count match the prior manifest;
- new resolved target root `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` does not exist before write.

Forbidden delete modes: glob, wildcard, `rm -rf`, directory deletion, recursive deletion, dataset-root deletion, run-id-prefix deletion, canonical/legacy deletion.

Post-delete checks required:

- both manifest objects absent;
- old run-id directory empty or absent;
- new resolved run-id directory not deleted by cleanup;
- catalog, published and canonical tree hashes unchanged.

This planning update executed no delete and no write. Future execution still requires an explicit runtime authorization that names this guard.
