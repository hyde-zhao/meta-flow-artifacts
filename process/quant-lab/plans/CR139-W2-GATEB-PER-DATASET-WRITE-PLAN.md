# CR139 W2 Gate B Per-Dataset Write Plan

## Scope

This plan is generated after user approval of `process/checkpoints/CR139-W2-GATEB-BATCH0-DECISION-BRIEF.md`.

Authorized now:

- Generate per-dataset write plan.
- Generate supporting evidence index and process state updates.

Still not authorized:

- Real lake writes.
- Catalog/provider-lake-catalog writes.
- Published pointer advance.
- Physical partition migration.
- Credential reads.
- NAS/QMT/MiniQMT/gateway runtime.
- Trading/small_live/live.
- Git remote write.

Operation counts for this planning step are all zero for provider fetch, lake write, catalog write, pointer publish, physical migration, credential read, NAS operation, runtime operation, and Git remote write.

## Inputs

| Input | Path |
|---|---|
| Gate A aligned inventory index | `process/evidence/CR139-W2-GATEA-REAL-LAKE-INVENTORY-DATALAKE-2026-06-29.index.json` |
| Gate B Batch 0 preflight index | `process/evidence/CR139-W2-GATEB-BATCH0-PREFLIGHT-2026-06-29.index.json` |
| Gate B Batch 0 duplicate profile index | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE.index.json` |
| Approved Decision Brief | `process/checkpoints/CR139-W2-GATEB-BATCH0-DECISION-BRIEF.md` |
| Dry-run execution plan | `process/plans/CR139-W2-DATA-LAKE-DRY-RUN-EXECUTION-PLAN.md` |

## Global Preconditions Before Any Write

| ID | Status | Requirement |
|---|---|---|
| GB-PRE-001 | NOT_SATISFIED_FOR_WRITE | Establish a fenced/quiescent window and re-run Gate A inventory on `/home/hyde/data/quant-lab/data-lake` immediately before the write batch, or record explicit drift risk acceptance. |
| GB-PRE-002 | NOT_SATISFIED_FOR_WRITE | For every dataset to be written, complete a per-dataset full-row exact duplicate profile or record explicit N/A with reason. |
| GB-PRE-003 | PLANNED_ONLY | Write mode must be additive-only to new CR139 run-id candidate paths; in-place overwrite, delete, or append into legacy paths is forbidden. |
| GB-PRE-004 | PLANNED_ONLY | Generate object manifest, pre-write snapshot, post-write snapshot, row/hash evidence, and rollback manifest for each approved write batch. |
| GB-PRE-005 | NOT_AUTHORIZED | Gate C catalog/checksum metadata, Gate D pointer advance, and Gate E physical migration remain separate gates and are not part of Gate B write execution. |

## Path Contract

Run IDs must be built with `market_data.lake_layout.build_cr139_run_id(dataset=<dataset>, source=legacy_lake, as_of_date=2026-06-29, purpose=canonical)`.

Candidate path template:

```text
candidate/parquet/dataset=<dataset>/schema_version=1.0/run_id=<cr139_run_id>
```

Quarantine path template for duplicate-key or events datasets:

```text
candidate/parquet/dataset=<dataset>/schema_version=1.0/run_id=<cr139_run_id>/quarantine
```

Legacy canonical paths are read-only inputs. Gate B must not update catalog paths, provider-lake-catalog paths, published pointers, or legacy physical partitions.

## Batch Strategy

| Batch | Status | Dataset Set | Rule |
|---|---|---|---|
| Batch 0 | Complete preflight only | `adj_factor`, `liquidity_capacity`, `market_cap`, `prices`, `prices_limit`, `trade_status`, `events` | No writes. Evidence only. |
| Batch 1A | Partially written and verified | `bse_code_mapping`, `lifecycle_code_change`, `hs300_index`, `industry_classification`, `prices_limit_code_change_fixes`, `prices_limit_coverage_exclusions` | Small/static datasets with no full-row duplicates. Candidate writes are verified for `bse_code_mapping` and safe-5; catalog/pointer/migration remain unauthorized. |
| Batch 1A-exact-dedup | Candidate writes verified | `trade_calendar`, `stock_basic` | Small/static datasets with exact full-row duplicates but no duplicate-key metadata conflict. Deduplicated candidate writes are verified; catalog/pointer/migration remain unauthorized. |
| Batch 1B | Ready for write authorization review after blocker planning | `index_members`, `index_weights` | `index_members` requires exact full-row dedup; `index_weights` must preserve mixed row-level PIT status and remain separate from membership semantics. |
| Batch 2 | Blocked pending deeper review | `liquidity_capacity`, `market_cap`, `events`, `trade_status`, `adj_factor`, `prices`, `prices_limit` | One dataset per authorization after full-row profile and dedup/quarantine review. |

## Per-Dataset Plan

| Dataset | Batch | Rows | Duplicate Rows Over Unique | Planned Action | Candidate Run ID | Write Blockers |
|---|---|---:|---:|---|---|---|
| `bse_code_mapping` | Batch 1A | 248 | 0 | Candidate write completed and verified | `cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `lifecycle_code_change` | Batch 1A | 250 | 0 | Candidate write completed and verified | `cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `hs300_index` | Batch 1A | 11,511 | 0 | Candidate write completed and verified | `cr139-w2-hs300_index-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `industry_classification` | Batch 1A | 11,049 | 0 | Candidate write completed and verified | `cr139-w2-industry_classification-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `trade_calendar` | Batch 1A-exact-dedup | 10,952 | 373 | Dedup candidate write completed and verified: retained 10,579, dropped 373 | `cr139-w2-trade_calendar-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `prices_limit_code_change_fixes` | Batch 1A | 1,129 | 0 | Candidate write completed and verified | `cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `prices_limit_coverage_exclusions` | Batch 1A | 91,018 | 0 | Candidate write completed and verified | `cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `stock_basic` | Batch 1A-exact-dedup | 55,166 | 11,608 | Dedup candidate write completed and verified: retained 43,558, dropped 11,608 | `cr139-w2-stock_basic-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `index_members` | Batch 1B-exact-dedup | 295,800 | 12,300 | Exact-dedup candidate write after authorization: retain 283,500, drop 12,300 exact duplicates | `cr139-w2-index_members-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized |
| `index_weights` | Batch 1B-mixed-pit | 283,800 | 0 | Mixed-PIT preserving candidate copy after authorization | `cr139-w2-index_weights-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized; preserve row-level PIT status |
| `liquidity_capacity` | Batch 2 | 17,095,488 | 1,830 | Candidate write verified: main 17,091,828 rows + quarantine 3,660 rows | `cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical` | catalog/pointer/migration not authorized; conflict truth unresolved |
| `market_cap` | Batch 2 | 17,108,103 | 106,776 | One-dataset Batch 2 with quarantine path | `cr139-w2-market_cap-legacy_lake-20260629-canonical` | metadata-conflict quarantine; fenced Gate A rerun; full-row profile; dataset authorization |
| `events` | Batch 2 | 362,887 | 11,360 | Events quarantine candidate; readiness rows = 351,527 | `cr139-w2-events-legacy_lake-20260629-canonical` | duplicate primary-key quarantine; fenced Gate A rerun; full-row profile; dataset authorization |
| `trade_status` | Batch 2 | 21,569,907 | 3,920,399 | One-dataset Batch 2 with quarantine path | `cr139-w2-trade_status-legacy_lake-20260629-canonical` | metadata-conflict quarantine; fenced Gate A rerun; full-row profile; dataset authorization |
| `adj_factor` | Batch 2 | 29,970,830 | 12,064,870 | One-dataset Batch 2 with quarantine path | `cr139-w2-adj_factor-legacy_lake-20260629-canonical` | metadata-conflict quarantine; fenced Gate A rerun; full-row profile; dataset authorization |
| `prices` | Batch 2 | 28,646,730 | 11,553,072 | One-dataset Batch 2 with quarantine path | `cr139-w2-prices-legacy_lake-20260629-canonical` | metadata-conflict quarantine; fenced Gate A rerun; full-row profile; dataset authorization |
| `prices_limit` | Batch 2 | 40,962,525 | 21,252,721 | One-dataset Batch 2 with quarantine path | `cr139-w2-prices_limit-legacy_lake-20260629-canonical` | metadata-conflict quarantine; fenced Gate A rerun; full-row profile; dataset authorization |

## Batch 1A Exact Full-Row Duplicate Datasets

`trade_calendar` and `stock_basic` were moved out of direct Batch 1A copy-write after the Batch 1A remaining readiness check found exact full-row duplicates under the Gate A inventory source scope (`physical_canonical_root`). These are not Batch 2 metadata-conflicting duplicate-key datasets; they require a smaller exact-dedup candidate planning gate.

| Dataset | Rows | Unique Full Rows | Exact Duplicate Groups | Rows Over Unique | Required Next Step |
|---|---:|---:|---:|---:|---|
| `trade_calendar` | 10,952 | 10,579 | 373 | 373 | Read-only exact-dedup candidate planning |
| `stock_basic` | 55,166 | 43,558 | 6,129 | 11,608 | Read-only exact-dedup candidate planning |

Exact-dedup planning and candidate writes are complete. The writes produced one deduplicated candidate parquet object per dataset and did not authorize catalog, pointer, or physical migration changes.

Retain rule: sort by `_cr139_source_file ASC`, `_cr139_source_row_number ASC`, then drop exact full-row duplicates keeping first. Because duplicate rows are exact copies across all data columns, no quarantine is recommended.

| Dataset | Candidate Shape | Planning Evidence |
|---|---|---|
| `trade_calendar` | one parquet under `candidate/parquet/dataset=trade_calendar/schema_version=1.0/run_id=cr139-w2-trade_calendar-legacy_lake-20260629-canonical` | `process/evidence/CR139-W2-GATEB-BATCH1A-EXACT-DEDUP-WRITE-EXECUTION.index.json` |
| `stock_basic` | one parquet under `candidate/parquet/dataset=stock_basic/schema_version=1.0/run_id=cr139-w2-stock_basic-legacy_lake-20260629-canonical` | `process/evidence/CR139-W2-GATEB-BATCH1A-EXACT-DEDUP-WRITE-EXECUTION.index.json` |


## Batch 1B Blocker Planning

Batch 1B readiness found two blockers and they are now planned explicitly:

| Dataset | Updated Batch | Source Rows | Retained Rows | Dropped Rows | Planned Action | Evidence |
|---|---|---:|---:|---:|---|---|
| `index_members` | Batch 1B-exact-dedup | 295,800 | 283,500 | 12,300 | Additive exact full-row dedup candidate write after explicit authorization | `process/evidence/CR139-W2-GATEB-BATCH1B-BLOCKER-PLANNING.index.json` |
| `index_weights` | Batch 1B-mixed-pit | 283,800 | 283,800 | 0 | Additive candidate copy preserving row-level mixed PIT status after explicit authorization | `process/evidence/CR139-W2-GATEB-BATCH1B-BLOCKER-PLANNING.index.json` |

`index_members` retain rule: sort by `_cr139_source_file ASC`, `_cr139_source_row_number ASC`, then drop exact full-row duplicates keeping first. No quarantine is recommended because dropped rows are exact copies of retained rows.

`index_weights` PIT decision: preserve row-level `pit_status` values (`pit_available=283,500`, `pit_incomplete=300`); do not collapse the dataset-level truth to fully PIT available, and do not treat `index_weights` as a substitute for `index_members`.

The 300 `pit_incomplete` rows are intentionally kept in the candidate with their row-level status. Gate B does not decide whether those rows are publishable current truth. Before any future Gate D pointer advance for `index_weights`, a separate decision must choose publish, isolate, repair, or waive handling for those 300 rows.

Batch 1B write atomicity policy: fail-stop, no automatic rollback. If one object is written and the next object fails, stop remaining writes, emit partial-completion evidence plus rollback manifest, and wait for a separate rollback or continue decision. Automatic deletion of written candidate objects is not authorized by the Batch 1B write gate.

Batch 1B writes remain unauthorized until a separate write authorization gate approves exactly these two candidate actions. Catalog, pointer, physical migration, NAS/runtime/credential/Git, and rollback remain outside Gate B.

## Duplicate-Key Datasets

Deterministic dedup policy for D-GATEB-002:

- Default for metadata-conflicting duplicate keys: quarantine, not dedup.
- Deterministic dedup may be proposed only when the per-dataset full-row exact duplicate profile proves the duplicate-key group is metadata-compatible, the dataset owner approves the sort key as business truth, and all required sort fields are present.
- Required sort key:

```sql
ORDER BY source_run_id DESC, available_at DESC, lineage_raw_checksum DESC
```

- Rationale: the ordering is deterministic and prefers the newest source run, then newest availability timestamp, then stable lineage checksum as a tie-breaker. It is not sufficient for metadata-conflicting groups; those remain quarantine by default under D-GATEB-001. If any required sort field is absent, deterministic dedup is not allowed without a new reviewed sort policy.
- Forbidden fallback: implicit keep-last or physical file order.

| Dataset | Duplicate Key Groups | Duplicate Rows In Groups | Rows Over Unique | Metadata Conflict Signal |
|---|---:|---:|---:|---|
| `liquidity_capacity` | 1,830 | 3,660 | 1,830 | source_run_id/lineage conflicts: 1,830 |
| `market_cap` | 106,776 | 213,552 | 106,776 | source_run_id/lineage conflicts: 106,776 |
| `events` | 11,357 | 22,717 | 11,360 | source_run_id/lineage conflicts: 11,357 |
| `trade_status` | 3,762,192 | 7,682,591 | 3,920,399 | source_run_id/lineage conflicts: 3,762,192; available_at conflicts: 74,956 |
| `adj_factor` | 11,824,907 | 23,889,777 | 12,064,870 | source_run_id/lineage conflicts: 81,852; available_at conflicts: 74,960 |
| `prices` | 11,313,190 | 22,866,262 | 11,553,072 | source_run_id/lineage conflicts: 81,788 |
| `prices_limit` | 17,356,374 | 38,609,095 | 21,252,721 | source_run_id/lineage conflicts: 7,445,795; available_at conflicts: 74,956 |

## Rollback Model For Future Write

Allowed rollback: delete only objects listed in the Gate B object manifest for the newly written CR139 run-id candidate path.

Forbidden rollback:

- Delete legacy canonical objects.
- Edit catalog pointers.
- Advance or rewind published pointer.
- Mutate physical partitions outside the new candidate path.

If any written object is absent from the object manifest, the Gate B attempt is failed and must be replanned from Gate A/B evidence.

## Gate C / D / E Boundary

| Gate | Status In This Plan | Rule |
|---|---|---|
| Gate C checksum/catalog | Plan only | Checksum backfill plan is required before catalog or pointer readiness. No catalog write is authorized here. |
| Gate D published pointer | Not authorized | This CR stage does not advance any dataset to published. Gate D requires independent authorization. |
| Gate E physical migration | Plan only | This plan records the decision surface: full 17 dataset migration vs publish-target-only + waiver. No physical migration is authorized here. |

## Recommended Next Review

Review this plan as `Gate B per-dataset write plan review`.

Recommended review decision:

- Approve only the next explicitly named pre-write action or write batch.
- Do not blanket-authorize all datasets.
- If choosing a first real write batch, use Batch 1A only after fenced Gate A rerun and per-dataset full-row profile evidence.

## Batch 2 Refined Risk Buckets

Batch 2 deep profile review refined the initial conservative `7/7 quarantine` routing. The table below keeps the raw profile counts and adds a planning bucket derived as `duplicate-key over-unique - full-row fingerprint duplicate over-unique`. This is a routing estimate; each future dataset write gate must materialize the exact buckets and run pre/post-write content validation before writing.

| Dataset | Dup-key over unique | Full-row dup over unique | Estimated non-identical same-key conflicts | Conflict share | Risk class | Refined action |
|---|---:|---:|---:|---:|---|---|
| `liquidity_capacity` | 1,830 | 0 | 1,830 | 100.0% | `pure_metadata_conflict` | `quarantine_metadata_conflict_key_groups_only` |
| `market_cap` | 106,776 | 0 | 106,776 | 100.0% | `pure_metadata_conflict` | `quarantine_metadata_conflict_key_groups_only` |
| `events` | 11,360 | 0 | 11,360 | 100.0% | `primary_key_conflict_quarantine` | `quarantine_duplicate_primary_key_groups_only` |
| `trade_status` | 3,920,399 | 79,870 | 3,840,529 | 98.0% | `mostly_metadata_conflict_with_required_exact_bucket_decision` | `quarantine_metadata_conflict_key_groups_and_exact_dedup_proven_exact_copy_bucket` |
| `adj_factor` | 12,064,870 | 11,907,842 | 157,028 | 1.3% | `mostly_exact_duplicates_with_small_conflict_bucket` | `exact_dedup_main_bucket_plus_quarantine_conflict_bucket` |
| `prices` | 11,553,072 | 11,470,750 | 82,322 | 0.7% | `mostly_exact_duplicates_with_small_conflict_bucket` | `exact_dedup_main_bucket_plus_quarantine_conflict_bucket` |
| `prices_limit` | 21,252,721 | 13,728,564 | 7,524,157 | 35.4% | `mixed_exact_duplicates_and_metadata_conflicts` | `exact_dedup_bucket_plus_quarantine_conflict_bucket` |

Machine consumer note: `conflict_share_of_duplicate_key_excess` is populated for all seven Batch 2 datasets as an alias for the estimated conflict share. `trade_status` exact-copy bucket handling is now required; exact-dedup is recommended for the 79,870 proven exact-copy rows while metadata-conflicting groups remain quarantine.

Additional rule for `prices`: source files have schema variants; its future write authorization must name a schema-normalization strategy before candidate write. Valid options must be reviewed explicitly, such as union schema with null fill, a reviewed canonical subset, or a design delta.



### Batch 2 liquidity_capacity Write Closure

`liquidity_capacity` candidate write completed after explicit Gate B authorization. The write produced one main candidate parquet object with 17,091,828 non-conflict rows and one quarantine parquet object with 3,660 metadata-conflict duplicate-key rows. No catalog, pointer, published, canonical, NAS, runtime, credential, Git remote, physical migration, or rollback action was authorized or executed. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION.index.json`.


### Batch 2 liquidity_capacity Resolution Planning

Read-only resolution planning is complete for `liquidity_capacity`. All 1,830 duplicate-key groups are metadata-only conflicts: business columns are identical, while `source_run_id` and `lineage_raw_checksum` differ between `run-cr034-chapter3-backfill-2000` and `run-cr034-chapter3-smoke-20000104-20000105`. The recommended resolved candidate rule is to select the full-year backfill row and quarantine the smoke-window row as superseded evidence.

Future resolved candidate proposal, not yet written:

| Object | Relative path | Rows |
|---|---|---:|
| resolved main candidate | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/part-liquidity-capacity.parquet` | 17093658 |
| superseded smoke quarantine | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/quarantine/part-liquidity-capacity-superseded-smoke.parquet` | 1830 |

Current quarantine-only candidate remains untouched. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-RESOLUTION-PLANNING.index.json`.


### Batch 2 Pure Metadata Conflict Batch Planning

Read-only batch planning is complete for `liquidity_capacity` and `market_cap`. Both are candidates for a single future resolved write batch using the same rule: select `run-cr034-chapter3-backfill-2000` over `run-cr034-chapter3-smoke-20000104-20000105` when business columns are identical, and quarantine the superseded smoke rows as evidence.

| Dataset | Future main rows | Future quarantine rows | Future run_id | Status |
|---|---:|---:|---|---|
| `liquidity_capacity` | 17,093,658 | 1,830 | `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` | `pass_resolution_plan_ready_for_write_authorization_review` |
| `market_cap` | 17,001,327 | 106,776 | `cr139-w2-market_cap-legacy_lake-20260629-resolved` | `blocked_resolution_issues` |


Recommended next write authorization, not yet granted: `authorize Gate B Batch 2 pure metadata resolved candidate writes only for liquidity_capacity and market_cap`. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-RESOLUTION-PLANNING.index.json`.


#### Batch 2 market_cap precedence decision update

The pure metadata batch planning scan found that `market_cap` is metadata-only, but it has two source-pair buckets rather than the single backfill-vs-smoke pattern seen in `liquidity_capacity`:

| Source pair | Groups | Recommended selection | Decision status |
|---|---:|---|---|
| `run-cr034-chapter3-backfill-2000` vs `run-cr034-chapter3-smoke-20000104-20000105` | 1,830 | select backfill, quarantine smoke | covered by existing pattern |
| `run-stage3-data-update-20260530-20260626-aux-20260627` vs `run-stage3-data-update-20260530-20260626-prices-20260627` | 104,946 | recommend select aux, quarantine prices | `D-GATEB-B2-MARKETCAP-001` required |

Therefore `liquidity_capacity + market_cap` combined resolved write is not ready until `D-GATEB-B2-MARKETCAP-001` is approved. `liquidity_capacity` can still be authorized alone, or the combined batch can proceed after the market_cap source precedence decision is accepted. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-RESOLUTION-PLANNING.index.json`.


### Batch 2 Strategy Unification Blocker

Quality review found a real strategy conflict before any pure metadata batch write:

| Dataset | Existing candidate | Existing strategy | New recommended strategy | Impact |
|---|---|---|---|---|
| `liquidity_capacity` | `cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical` | main 17,091,828 + quarantine 3,660; full conflict group quarantine | `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` main 17,093,658 + quarantine 1,830; select business-identical backfill row to main | Existing main is missing 1,830 keys; keeping it while writing market_cap resolved would create inconsistent candidate semantics. |

Required decisions before batch write:

| Decision ID | Recommended | Alternative | Why it matters |
|---|---|---|---|
| `D-GATEB-B2-LIQUIDITY-001` | Roll back exactly two old `liquidity_capacity` `cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical` candidate objects and write `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` | Keep old quarantine-only candidate and apply resolved strategy only to market_cap | Recommendation keeps main candidate complete and avoids Gate D publish gaps. |
| `D-GATEB-B2-MARKETCAP-001` | Select `run-stage3-data-update-20260530-20260626-aux-20260627` and quarantine `run-stage3-data-update-20260530-20260626-prices-20260627` for 104,946 aux-vs-prices metadata-only groups | Select prices run and quarantine aux run | Business values are identical; decision affects lineage ownership. |

Fastest accurate route after approval: one explicit compound Gate B execution that deletes only the two old `liquidity_capacity` candidate objects listed in its object manifest, writes `liquidity_capacity` resolved candidate, and writes `market_cap` resolved candidate. Expected operations: candidate object deletes = 2, lake writes = 4, catalog/pointer/migration/NAS/runtime/credential/Git = 0.


#### Batch 2 liquidity delete guard adopted

User review approved `D-GATEB-B2-MARKETCAP-001` and accepted `D-GATEB-B2-LIQUIDITY-001` option A with an execution guard. The guard is mandatory for any future strategy-unification execution:

- delete only the two exact paths listed in `process/evidence/CR139-W2-GATEB-BATCH2-LIQUIDITY-CAPACITY-WRITE-EXECUTION-2026-06-29T184303+0800-OBJECT-MANIFEST.json`;
- validate each path is under `candidate/`, contains exact run id `cr139-w2-liquidity_capacity-legacy_lake-20260629-canonical`, has no wildcard/glob characters, and matches prior manifest sha256/size/row_count before deletion;
- forbid glob, wildcard, `rm -rf`, directory deletion, dataset-root deletion, run-id-prefix deletion, and any `canonical/`, `published/`, `catalog/`, or legacy deletion;
- after deletion verify old run dir empty/absent, new `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` target not deleted, and catalog/published/canonical hashes unchanged.

Future execution is ready for explicit runtime authorization with this guard, but no delete or write was executed by this planning update.


### Batch 2 Pure Metadata Strategy-Unification Execution Closure

Execution completed for `liquidity_capacity` and `market_cap` with `GATEB-B2-LIQUIDITY-DELETE-GUARD-001`. Exactly two old `liquidity_capacity` candidate objects were deleted by prior manifest path, then four resolved candidate objects were written. Catalog, published, canonical, physical migration, NAS, runtime, credential, Git remote, and legacy data were not changed.

| Dataset | Resolved main rows | Superseded quarantine rows | Status |
|---|---:|---:|---|
| `liquidity_capacity` | 17,093,658 | 1,830 | verified |
| `market_cap` | 17,001,327 | 106,776 | verified |

Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-STRATEGY-UNIFICATION-EXECUTION.index.json`.


### Batch 2 Remaining Split Planning Closure

Read-only split planning completed for `adj_factor`, `prices`, `prices_limit`, `events`, and `trade_status`. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-REMAINING-SPLIT-PLANNING.index.json`. No lake write, candidate delete, catalog write, pointer advance, physical migration, NAS, runtime, credential read, or Git remote write was executed.

| Dataset | Source rows | Duplicate-key groups | Exact-copy groups / rows | Metadata-only groups / rows | Business-conflict groups / rows | Planned main rows | Planned quarantine rows | Planned exact-copy drops | Recommended write mode |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| `adj_factor` | 29,970,830 | 11,824,907 | 11,743,055 / 23,486,110 | 81,852 / 403,667 | 0 / 0 | 17,905,960 | 321,815 | 11,743,055 | `metadata-only resolved candidate planning` |
| `prices` | 28,646,730 | 11,313,190 | 11,231,402 / 22,462,804 | 1,830 / 3,660 | 79,958 / 399,798 | 17,013,700 | 401,628 | 11,231,402 | `split exact-dedup plus metadata resolution plus business-conflict quarantine planning` |
| `prices_limit` | 40,962,525 | 17,356,374 | 9,910,579 / 19,821,158 | 6,946,008 / 17,526,841 | 499,787 / 1,261,096 | 19,210,017 | 11,841,929 | 9,910,579 | `split exact-dedup plus metadata resolution plus business-conflict quarantine planning` |
| `events` | 362,887 | 11,357 | 0 / 0 | 10,583 / 21,166 | 774 / 1,551 | 350,753 | 12,134 | 0 | `primary-key-conflict quarantine planning` |
| `trade_status` | 21,569,907 | 3,762,192 | 0 / 0 | 70,087 / 140,512 | 3,692,105 / 7,542,079 | 13,957,403 | 7,612,504 | 0 | `split exact-dedup plus metadata resolution plus business-conflict quarantine planning` |

Planning implications:

- `adj_factor` is the only remaining Batch 2 dataset with zero business-conflict groups. It can be the next single-dataset write candidate after explicit approval of exact-copy dedup and metadata source precedence.
- `prices` remains blocked by schema normalization plus 79,958 business-conflict groups; it must not be written before the prices schema policy is approved.
- `prices_limit`, `events`, and `trade_status` have business-conflict buckets and require explicit per-dataset quarantine or semantic-resolution decisions before any write.
- These split buckets supersede the earlier coarse `duplicate-key over-unique - full-row fingerprint duplicate over-unique` routing estimate for these five datasets.


### Batch 2 adj_factor Write Closure

`adj_factor` additive-only candidate write completed and verified. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION.index.json`.

| Main rows | Quarantine rows | Dropped exact duplicates | Main duplicate rows over unique | Objects |
|---:|---:|---:|---:|---:|
| 17,905,960 | 321,815 | 11,743,055 | 0 | 2 |

Catalog, published pointer, canonical tree, physical migration, NAS/runtime/credential/Git remote, legacy deletion, and rollback remain unauthorized.


### Batch 2 Remaining Decision Plan

Decision plan completed after `adj_factor` write. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-REMAINING-DECISION-PLAN.index.json`.

| Dataset | Recommended next status |
|---|---|
| `events` | next explicit write candidate: split main/quarantine, business-conflict groups full quarantine |
| `prices` | blocked until union schema with null fill is approved |
| `prices_limit` | single-dataset split write gate only |
| `trade_status` | single-dataset split write gate only; keep separate due high conflict count |


### Batch 2 events Write Closure

`events` additive-only split candidate write completed and verified. Evidence index: `process/evidence/CR139-W2-GATEB-BATCH2-EVENTS-WRITE-EXECUTION.index.json`.

| Main rows | Quarantine rows | Metadata-only groups | Business-conflict groups | Main duplicate rows over unique | Objects |
|---:|---:|---:|---:|---:|---:|
| 350,753 | 12,134 | 10,583 | 774 | 0 | 2 |

Catalog, published pointer, canonical tree, physical migration, NAS/runtime/credential/Git remote, legacy deletion, and rollback remain unauthorized.


### Batch 2 Remaining Large Table Write Closure

`prices`, `prices_limit`, and `trade_status` additive-only split candidate writes completed and verified.

| Dataset | Main rows | Quarantine rows | Dropped exact duplicates | Business-conflict groups | Main duplicate rows over unique | Evidence |
|---|---:|---:|---:|---:|---:|---|
| `prices` | 17,013,700 | 401,628 | 11,231,402 | 79,958 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-PRICES-WRITE-EXECUTION.index.json` |
| `prices_limit` | 19,210,017 | 11,841,929 | 9,910,579 | 499,787 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-PRICES_LIMIT-WRITE-EXECUTION.index.json` |
| `trade_status` | 13,957,403 | 7,612,504 | 0 | 3,692,105 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-TRADE_STATUS-WRITE-EXECUTION.index.json` |

All Batch 2 Gate B candidate writes are now complete. Catalog, provider catalog/provider-lake-catalog, published pointer advance, physical migration, NAS/runtime/credential/Git remote, legacy deletion, and rollback deletion remain unauthorized.
