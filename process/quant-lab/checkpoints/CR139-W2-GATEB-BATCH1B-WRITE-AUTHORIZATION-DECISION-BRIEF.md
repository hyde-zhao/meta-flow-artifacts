# CR139 W2 Gate B Batch 1B Write Authorization Decision Brief

Generated at: 2026-06-29T16:18:20+08:00

## Approver Summary

Batch 1B blocker planning completed in read-only mode. It did not write lake objects, catalog, pointers, physical partitions, credentials, NAS, runtime, Git remote, or rollback objects.

Recommended decision: authorize only the two planned Batch 1B candidate writes:

- `index_members`: additive exact full-row dedup candidate write, retaining 283,500 rows and dropping 12,300 exact duplicate rows.
- `index_weights`: additive candidate copy preserving all 283,800 rows and row-level mixed PIT status.

Important semantic boundary: the `index_weights` candidate will include 300 rows whose row-level `pit_status` is `pit_incomplete`. Gate B candidate write preserves those rows and their status; it does not decide whether those 300 rows are publishable current truth. Before any future Gate D published pointer advance for `index_weights`, a separate review must decide whether to publish, isolate, repair, or waive those 300 `pit_incomplete` rows.

This does not authorize Batch 2, catalog writes, published pointer advance, physical migration, NAS/runtime/credential/Git, or rollback deletion.

## Evidence Summary

| Item | Result |
|---|---|
| Planning status | pass_with_risk_ready_for_batch1b_write_authorization_review |
| `index_members` source rows | 295,800 |
| `index_members` retained rows | 283,500 |
| `index_members` dropped exact duplicate rows | 12,300 |
| `index_members` exact duplicate groups | 6,300 |
| `index_weights` source/retained rows | 283,800 |
| `index_weights` PIT distribution | `pit_available=283,500`, `pit_incomplete=300` |
| `index_weights` Gate D precondition | 300 `pit_incomplete` rows require separate publish/isolate/repair/waiver decision before pointer advance |
| Operation counts | all 0 |

Evidence refs:

- Blocker planning index: `process/evidence/CR139-W2-GATEB-BATCH1B-BLOCKER-PLANNING.index.json`
- Blocker planning detail: `process/evidence/CR139-W2-GATEB-BATCH1B-BLOCKER-PLANNING-2026-06-29T161820+0800.json`
- Blocker planning check: `process/checks/CR139-W2-GATEB-BATCH1B-BLOCKER-PLANNING-2026-06-29.md`
- Prior readiness index: `process/evidence/CR139-W2-GATEB-BATCH1B-PREWRITE-READINESS.index.json`
- Updated write plan JSON: `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.json`
- Updated write plan MD: `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.md`

## Decision Collection Coverage

| Source scanned | Candidate issue groups | Included decision items | Duplicates merged | N/A |
|---|---:|---:|---:|---:|
| Batch 1B blocker planning detail/index | 5 | 5 | 0 | 0 |
| Updated write plan | 3 | 3 | 0 | 0 |
| Prior readiness blockers | 2 | 2 | 0 | 0 |

All 10 candidate issue groups are represented below as 6 decision items.

## Pending Human Decisions

| ID | Type | Question | Recommended | Alternative | Risk / Impact |
|---|---|---|---|---|---|
| D-B1B-WRITE-001 | runtime_authorization | Should the next write gate include both Batch 1B datasets? | Yes, authorize only `index_members` exact-dedup and `index_weights` mixed-PIT preserving candidate writes. | Authorize only one dataset first. | Both are planned and small enough for one narrow batch, but evidence must remain per-object. |
| D-B1B-WRITE-002 | implementation | How should `index_members` duplicates be handled? | Apply exact full-row dedup: retain first deterministic source-file/source-row copy, drop exact duplicates, no quarantine. | Preserve duplicates as-is. | Preserving 12,300 duplicates weakens CR139 write dedup guarantees. |
| D-B1B-WRITE-003 | architecture | How should `index_weights` mixed PIT status be handled? | Preserve row-level mixed PIT status and explicitly avoid dataset-level fully-PIT claim. | Block `index_weights` until all rows are PIT available. | Blocking may be too conservative; silently collapsing mixed PIT to fully available is unsafe. |
| D-B1B-WRITE-004 | risk_acceptance | How should operator-fence risk be handled for the write? | Accept PASS_WITH_RISK for the narrow candidate write unless mechanical fence proof is supplied. | Require mechanical external writer lock before write. | Without mechanical lock, concurrent external write risk is accepted by operator. |
| D-B1B-WRITE-005 | implementation | What batch atomicity policy applies to this two-object write? | Use fail-stop, no automatic rollback: if one object writes and the next fails, stop, emit partial evidence and rollback manifest, then wait for separate rollback or continue decision. | Auto-rollback written objects on failure. | Auto-rollback would be an implicit lake deletion and should remain separately authorized. |
| D-B1B-WRITE-006 | runtime_authorization | What remains not authorized? | Batch 2, catalog/provider/provider-lake-catalog writes, pointer advance, physical migration, credential read, NAS/runtime/QMT/MiniQMT/gateway/trading/live, Git remote, rollback deletion. | Blanket-authorize downstream gates. | Blanket authorization would mutate current truth or rollback scope without review. |

## Expected Write Scope If Approved

Allowed:

- Create one candidate parquet object for `index_members` under `candidate/parquet/dataset=index_members/schema_version=1.0/run_id=cr139-w2-index_members-legacy_lake-20260629-canonical` using exact-dedup retained rows only.
- Create one candidate parquet object for `index_weights` under `candidate/parquet/dataset=index_weights/schema_version=1.0/run_id=cr139-w2-index_weights-legacy_lake-20260629-canonical` preserving all rows and row-level PIT status.
- Generate per-dataset pre-write snapshot, post-write snapshot, object manifest, rollback manifest, zero-pointer-mutation proof, execution index, and execution check.
- Record batch execution order, per-object completion status, and the fail-stop/no-automatic-rollback policy in the execution index.

Gate D precondition for future publish:

- `index_weights` candidate includes `pit_incomplete=300` rows. Those rows must be explicitly reviewed before any published pointer advance. Gate B candidate write is not a publishability decision for them.

Forbidden:

- Any write outside those two candidate paths.
- Catalog/provider/provider-lake-catalog write.
- Published pointer advance.
- Physical partition migration.
- Credential read.
- NAS/runtime/QMT/MiniQMT/gateway/trading/live.
- Git remote write.
- Rollback deletion without separate rollback authorization.

Expected operation counts for the write execution if approved:

```json
{
  "provider_fetch": 0,
  "lake_write": 2,
  "catalog_write": 0,
  "current_pointer_publish": 0,
  "physical_partition_migration": 0,
  "credential_read": 0,
  "nas_operation": 0,
  "runtime_operation": 0,
  "git_remote_write": 0
}
```

## Exact Replies

- `approve` means: authorize only the two Batch 1B candidate writes described above. It does not authorize Batch 2, catalog, pointer, migration, NAS/runtime/credential/Git, or rollback deletion.
- `修改: <具体修改点>` means: change one or more decisions before write authorization.
- `reject` means: do not proceed to Batch 1B writes.

## Recommended Authorization Text

```text
authorize Gate B Batch 1B index_members exact-dedup candidate write and index_weights mixed-PIT-preserving candidate copy only
```
