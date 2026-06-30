# CR139 W2 Data Lake Dry-Run Execution Plan

## Decision Boundary

| Field | Value |
|---|---|
| Change | CR-139 |
| Batch | CR139-W2-DATA-CONTRACTS |
| Plan status | READY_FOR_REVIEW_AFTER_REVISION |
| Plan type | dry-run execution plan |
| Generated at | 2026-06-29T13:05:00+08:00 |
| Revised at | 2026-06-29T14:10:00+08:00 |
| Execution performed | No |
| Real lake write authorized | No |
| Provider catalog/provider-lake-catalog write authorized | No |
| Published pointer advance authorized | No |
| Physical partition migration authorized | No |

This plan is intentionally non-executing. It defines the object classes, order, evidence, and gates needed before any real lake mutation. It does not read credentials and does not touch runtime, NAS, QMT, MiniQMT, gateway, trading/small_live/live, or Git remote write paths.

## Revision Decisions

| Review Item | Decision | Rationale |
|---|---|---|
| Physical migration vs pointer advance order | Path-changing Gate E must complete before Gate D. If migration changes catalog/pointer/manifest path refs, Gate C must be re-authorized after Gate E and before Gate D. | Catalog and pointer payloads include physical path fields such as `canonical_path`, `published_path`, and manifest `candidate_path`; advancing pointer before migration can leave current truth pointing at old paths. |
| Gate A no-write guarantee | `--out` paths must resolve outside `lake_root`, and tooling must fail fast when they do not. | Gate A is only safe if report output cannot be written into the lake tree. |
| Lake root basename consistency | The local lake root used for Gate B must end in `data-lake`, matching the NAS final directory `/data-lake`. A legacy local root ending in `lake` can be used only as historical/read-only evidence and cannot directly authorize Gate B. | Local and NAS lake roots must stay path-compatible before real writes, catalog refs, and migration plans are generated. |
| A-to-B transition | "Clean" means gateable and complete, not "zero findings." | CR139 expects findings; the decision threshold must separate expected remediation findings from blocking inventory uncertainty. |
| Real lake freshness | Gate A inventory is valid for 24 hours only, and any known lake write/fence break invalidates it. | B/C/D evidence must be based on a current object graph. |
| Gate B write mode | Additive-only writes to new CR139 run-id paths; no in-place overwrite/delete/append into existing partition paths. | Rollback by deleting newly written objects is only safe for non-overlapping new paths. |
| Gate B batching | Gate B is split into Batch 0 preflight-only, Batch 1 low-risk zero-duplicate small datasets, and Batch 2 large/duplicate-key datasets after dedup decision review. | 48,899,668 duplicate keys are a business-data decision surface and must not be resolved during a write step. |
| Gate E scope decision | Before Gate C/D, choose full 17-dataset path migration to CR139 naming or publish-target-only migration with explicit waivers for legacy paths. | Gate A found 17/17 catalog paths are non-CR139 named and 17/17 datasets are unpublished. |

## Current Proof Before Lake Touch

| Proof | Status | Evidence |
|---|---|---|
| W2 code contracts implemented | PASS | 9 CP6 Story return/evidence/check artifacts |
| W2 fixture/static verification | PASS_WITH_RISK | `docs/quality/VERIFICATION-REPORT-CR139-W2-DATA-CONTRACTS.md` |
| Integrated quality review | PASS_WITH_RISK | `docs/quality/REVIEW-CR139-W2-DATA-CONTRACTS.md` |
| Real lake object enumeration | PASS_WITH_RISK | Gate A no-write inventory completed on legacy local root `/home/hyde/data/quant-lab/lake`; Gate B requires re-inventory on a root whose final path segment matches `data-lake` or an explicitly approved compatibility mount |
| Real mutation authorization | NOT_AUTHORIZED | Must be split by gate below |

## Planned Object Classes

| Object Class | Example Scope | Source Contract | Mutation Risk |
|---|---|---|---|
| Canonical partitions with legacy or non-conforming run-id names | canonical dataset partitions under data lake | CR139-S30 | Physical path mutation risk |
| Events partitions with schema gaps | event/audit-style canonical rows | CR139-S31 | Data repair/write risk |
| Canonical append candidates with duplicate keys | `(symbol, trade_date)` and dataset-specific key groups | CR139-S32 | Write rejection or quarantine risk |
| Catalog pointer records | current/published catalog records | CR139-S33, CR139-S39 | Current truth mutation risk |
| Manifest records | manifest metadata paired to catalog records | CR139-S33, CR139-S34 | Metadata write risk |
| Lineage checksum backfill candidates | existing manifest/catalog entries missing checksum | CR139-S34 | Metadata backfill risk |
| Incremental append plans | append-only candidate batches | CR139-S14 | Data write and pointer sequencing risk |
| Passive strategy evidence lineage | evidence index data_lineage fields | CR139-S22 | Consumer compatibility risk |

## Execution Order

| Step | Name | Mode Now | Gate Scope | Future Mutation Gate | Output Evidence |
|---:|---|---|---|---|---|
| 0 | No-write real lake inventory dry-run | Planned only | Gate A | None, but requires explicit read boundary and lake root input | Inventory report: datasets, partition counts, catalog refs, manifest refs, pointer snapshot |
| 1 | Schema freeze compatibility scan | Planned only | Gate A | None if read-only | Schema compatibility report by dataset |
| 2 | Run-id naming classification | Planned only | Gate A | Gate E only if rename/move is later proposed | Legacy/non-conforming path list |
| 3 | Events schema repair candidate plan | Planned only | Gate A planning, Gate B execution | Gate B before repaired data is written | Repair candidate diff and rejected rows |
| 4 | Write dedup preflight | Planned only | Gate A planning, Gate B execution | Gate B before append | Duplicate key report and append blocklist |
| 5 | Incremental append dry-run | Planned only | Gate A planning, Gate B execution | Gate B before append | Append plan with expected partitions and checksums |
| 6 | Physical partition migration dry-run | Planned only | Gate A planning, Gate E execution if needed | Gate E after backup/checksum proof and before Gate D | Move/copy/delete plan, rollback map |
| 7 | Catalog/manifest source-of-truth dry-run | Planned only | Gate A planning, Gate C execution | Gate C after final physical paths are known | Catalog/manifest pair diff |
| 8 | Lineage checksum backfill dry-run | Planned only | Gate A planning, Gate C execution | Gate C before metadata backfill | Checksum candidate table |
| 9 | Published pointer advance dry-run | Planned only | Gate A planning, Gate D execution | Gate D after Gate C and after all path-changing Gate E work is complete or waived | Pointer before/after proposal and rollback pointer |
| 10 | Post-mutation verification plan | Planned only | Each authorized gate | Runs after each authorized gate | Hash, row count, schema, catalog, pointer, and reader smoke evidence |

## Gate A: No-Write Inventory Dry-Run

| Item | Requirement |
|---|---|
| Authorized action | Read-only enumeration of configured real lake root and metadata files |
| Still forbidden | Writes, deletes, renames, provider calls, catalog writes, pointer advance, credentials, runtime |
| Required inputs | Lake root path or approved local mount whose final path segment is `data-lake`; proof that this root is the intended real production lake root or an explicitly approved read-only mirror; catalog configured root match or documented exception; explicit statement that no credentials need to be read; output path outside `lake_root`; expected quiescence/fence window |
| Required output | Inventory report with dataset list, partition count, pointer snapshot, manifest/catalog refs, suspected legacy run-id paths, and root basename consistency verdict |
| Stop condition | Missing lake root, local/NAS lake root basename mismatch without approved compatibility mount, lake root/catalog root mismatch without approved exception, credential required, unexpected provider/runtime access, inventory cannot prove no-write mode, or lake cannot be treated as quiescent/fenced for the reviewed window |

Gate A tooling must enforce the output boundary: `scripts/lake_inventory.py` and `scripts/lake_golden_baseline.py` fail fast when `--out` resolves inside `lake_root`, with the error `输出路径不能位于 lake_root 内: <path>`.

### Gate A Pass Threshold

Gate A may pass with findings. "Clean enough for Gate B" means:

| Criterion | Required Result |
|---|---|
| Inventory completeness | All expected catalog datasets, manifest refs, pointer refs, and physical partition counts are enumerated or missing items are explicitly classified. |
| Root authenticity | `lake_root` final path segment is `data-lake` and matches the configured catalog/lake root, or an explicit approved compatibility exception says this is a read-only mirror and not production. |
| No unexpected external dependency | No provider call, credential read, runtime, NAS/QMT/gateway, or remote write is required to enumerate the lake. |
| Object graph stability | Gate A snapshot is within the validity window and lake is quiescent/fenced, or any concurrent writes are listed and force a re-run before Gate B. |
| Finding classification | Expected CR139 findings such as S30 non-conforming paths, S31 schema gaps, or S32 duplicate keys are routed to their gates and do not by themselves block B; unknown datasets, unknown current pointers, catalog/path mismatches, or unclassified write requirements block B. |

## Gate B: Real Lake Write

| Item | Requirement |
|---|---|
| Authorized action | Write new candidate/canonical data objects only |
| Still forbidden unless separately approved | Catalog write, provider-lake-catalog write, pointer advance, physical rename/delete/move |
| Preconditions | Gate A inventory complete and valid on a `data-lake` root or approved compatibility mount, schema compatibility pass, dedup pass, repair plan reviewed, target paths are new CR139 run-id paths, target paths do not already exist, and the lake remains quiescent/fenced since Gate A or Gate A has been re-run |
| Required evidence | Pre-write hash/count snapshot, candidate write manifest, post-write hash/count snapshot, zero pointer mutation proof |
| Rollback | Remove only newly written candidate objects using the generated object manifest. Gate B does not touch catalog or pointer, so rollback must not require pointer operations; if rollback reveals a missing object-manifest entry, the Gate B attempt is failed and must be replanned from Gate A/B evidence. |
| Stop condition | Any write would append to, overwrite, delete from, or mutate an existing partition path; target run-id does not satisfy CR139 naming; pre-write snapshot is stale; or output cannot prove zero catalog/pointer mutation |

Gate B is additive-only. S14 incremental append and S31 events repair must materialize into new run-id paths; in-place overwrite or append into an existing physical partition is not allowed under Gate B.

### Gate B Batch Constraint

| Batch | Scope | Authorization Boundary | Required Decision/Evidence |
|---|---|---|---|
| Batch 0 | Six duplicate-key datasets plus `events`: `adj_factor`, `liquidity_capacity`, `market_cap`, `prices`, `prices_limit`, `trade_status`, `events` | Preflight-only; no lake object writes | Dedup report, repair candidate diff, quarantine/blocklist proposal, and reviewed keep/drop rule |
| Batch 1 | Low-risk zero-duplicate small datasets such as `bse_code_mapping`, `lifecycle_code_change`, `hs300_index`, `industry_classification`, `trade_calendar`, `prices_limit_code_change_fixes`, `prices_limit_coverage_exclusions` | Additive-only writes to new CR139 run-id paths after Gate B authorization | Per-dataset pre-write/post-write hash/count, object manifest, zero catalog/pointer mutation proof |
| Batch 2 | Large or duplicate-key datasets, including `prices`, `prices_limit`, `adj_factor`, `trade_status`, `market_cap`, `liquidity_capacity` | One dataset at a time after dedup decision review | Dataset-specific dedup decision, candidate manifest, rollback manifest, pre/post hash/count |

## Gate C: Catalog / Manifest Write

| Item | Requirement |
|---|---|
| Authorized action | Write catalog/manifest metadata records for already written or already existing data objects |
| Still forbidden unless separately approved | Published pointer/current truth advance, physical partition migration |
| Preconditions | Gate B complete when new data is involved; all path-changing Gate E work is complete or explicitly waived; source-of-truth diff reviewed; lineage checksum verified; catalog/manifest records point to final physical paths |
| Required evidence | Catalog/manifest before snapshot, diff, after snapshot, checksum validation |
| Rollback | Restore previous metadata files from before snapshot or write compensating supersession record if append-only metadata is required |

## Gate D: Published Pointer Advance

| Item | Requirement |
|---|---|
| Authorized action | Advance published/current pointer to reviewed catalog/manifest target |
| Still forbidden unless separately approved | Any physical partition migration or catalog path rewrite |
| Preconditions | Gate C complete after all path-changing Gate E work is complete or waived; no pending catalog/manifest path diff remains; pointer before/after proposal reviewed; reader smoke plan ready; rollback pointer verified |
| Required evidence | Pointer before snapshot, target catalog/manifest ref, pointer after snapshot, rollback pointer, post-advance reader smoke |
| Rollback | Repoint current pointer to previous published ref and rerun reader smoke |
| Stop condition | Any referenced `canonical_path`, `published_path`, `candidate_path`, manifest ref, or checksum would be invalidated by a later physical move; catalog/path refs are ambiguous; rollback pointer cannot be proven |

## Gate E: Physical Partition Migration

| Item | Requirement |
|---|---|
| Authorized action | Copy/move/rename/delete physical partitions according to reviewed migration map |
| Preconditions | Backup snapshot, checksum map, migration map, no current pointer has been advanced to paths that will be moved, and either catalog references are not yet pointing to the moved paths or a post-migration Gate C path refresh is explicitly planned |
| Required evidence | Pre-migration file list, checksum map, migration map, post-migration checksum, old/new path read smoke |
| Stop condition | Any checksum mismatch, unresolved duplicate path, missing backup, catalog ref ambiguity, or need to modify catalog/pointer refs without a separately authorized Gate C |
| Rollback | Restore from backup or reverse migration map; pointer must remain rollback-capable |

Gate E must not run after Gate D for any path referenced by the advanced pointer. If a post-pointer physical move is discovered, the safe route is: rollback pointer, run Gate E, run Gate C path refresh, then re-run Gate D.

## Validity Window

| Item | Rule |
|---|---|
| Gate A inventory validity | 24 hours from inventory completion, unless any known write/fence break occurs earlier |
| A-to-B gap | If Gate B starts after the validity window, if lake root basename changed, or if the prior inventory used `/home/hyde/data/quant-lab/lake`, re-run Gate A on the `data-lake` root or approved compatibility mount |
| A-to-C/D gap | Gate C and Gate D must cite the inventory snapshot and any post-Gate-B/E evidence; stale or inconsistent snapshots block the gate |
| Concurrent writes | Any unplanned write to lake/catalog/manifest/pointer during A-to-D invalidates the pending gate and requires re-inventory or explicit reconciliation |

## Review Recommendation

Do not combine Gates B, C, D, and E. The lowest-risk sequence is:

1. Approve Gate A only: no-write real lake inventory dry-run.
2. Review concrete inventory and object counts.
3. Before Gate B, confirm the production or mirror lake root uses a `data-lake` final path segment, establish a fence/quiescent window, and re-run Gate A on that root.
4. Run Gate B Batch 0 preflight-only and review dedup/repair decisions.
5. If Batch 0 evidence is acceptable, approve Gate B Batch 1 for low-risk additive-only candidate writes.
6. Approve Gate B Batch 2 one dataset at a time only after dedup decisions are reviewed.
7. Review data write evidence.
8. Decide Gate E scope: migrate all 17 datasets to CR139 naming, or migrate only publish targets and record explicit waivers for legacy paths.
9. If physical partition migration is necessary for paths that will be referenced by current truth, approve Gate E before Gate C/D.
10. Review migration evidence; stop on any checksum/path/catalog ambiguity.
11. Approve Gate C for catalog/manifest writes or path refresh to final physical paths.
12. Review metadata evidence.
13. Approve Gate D for published pointer advance only after path-changing Gate E is complete or waived.
14. Review reader smoke and rollback pointer.

## Explicit Non-Authorization

This plan is not approval to execute. It does not authorize real lake writes, provider catalog/provider-lake-catalog writes, published pointer advance, physical partition migration, credential reads, NAS/QMT/MiniQMT/gateway runtime, trading/small_live/live, or Git remote writes.
