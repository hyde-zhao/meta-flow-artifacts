# CR139 W2 Gate B Batch 1B Readiness Blocker Decision Brief

Generated at: 2026-06-29T16:07:13+08:00

## Approver Summary

Batch 1B consolidated pre-write readiness ran in read-only mode for:

- `index_members`
- `index_weights`

The readiness gate is **BLOCKED_NOT_READY_FOR_BATCH1B_WRITE_AUTHORIZATION_REVIEW**. No lake write, catalog write, pointer advance, physical migration, credential read, NAS/runtime operation, Git remote write, or rollback deletion was executed.

The readiness gate found two real blockers:

1. `index_members` has exact full-row duplicates: 295,800 rows, 283,500 unique full rows, 12,300 rows over unique across 6,300 duplicate groups.
2. `index_weights` has no full-row duplicates, but row-level `pit_status` contains both `pit_available` and `pit_incomplete`. Gate A inventory summarized the dataset as `pit_available`, so the aggregate inventory value is not sufficient for publish/readiness decisions.

Recommended decision: do not authorize Batch 1B writes yet. Start a read-only Batch 1B blocker planning gate:

- generate an exact-dedup candidate plan for `index_members`;
- generate a PIT semantics decision for `index_weights` that preserves mixed `pit_status` and explicitly states downstream reader implications;
- update the write plan after the planning gate, then return for a narrow write authorization review.

## Evidence Summary

| Item | Result |
|---|---|
| Gate A rerun drift count | 0 |
| Path collision count | 0 |
| Candidate targets existing before write | 0 |
| Operation counts | all 0 |
| `index_members` rows | 295800 |
| `index_members` unique full rows | 283500 |
| `index_members` exact duplicate groups | 6300 |
| `index_members` rows over unique | 12300 |
| `index_weights` rows | 283800 |
| `index_weights` unique full rows | 283800 |
| `index_weights` rows over unique | 0 |
| `index_weights` row-level PIT statuses | `pit_available`, `pit_incomplete` |

Evidence refs:

- Readiness index: `process/evidence/CR139-W2-GATEB-BATCH1B-PREWRITE-READINESS.index.json`
- Readiness detail: `process/evidence/CR139-W2-GATEB-BATCH1B-PREWRITE-READINESS-2026-06-29T160713+0800.json`
- Readiness check: `process/checks/CR139-W2-GATEB-BATCH1B-PREWRITE-READINESS-2026-06-29.md`
- Gate A rerun: `process/evidence/CR139-W2-GATEB-BATCH1B-PREWRITE-GATEA-2026-06-29T160000+0800.json`

## Decision Collection Coverage

| Source scanned | Candidate issue groups | Included decision items | Duplicates merged | N/A |
|---|---:|---:|---:|---:|
| Batch 1B readiness index/detail | 5 | 5 | 0 | 0 |
| Batch 1B readiness check | 4 | 4 | 0 | 0 |
| Gate A rerun vs row-level profile comparison | 2 | 2 | 0 | 0 |
| Existing Gate B write-plan rules | 3 | 3 | 0 | 0 |

All 14 candidate issue groups are represented below as 5 decision items. No item was excluded as N/A.

## Pending Human Decisions

| ID | Type | Question | Recommended | Alternative | Risk / Impact |
|---|---|---|---|---|---|
| D-B1B-BLOCK-001 | runtime_authorization | Should Batch 1B direct copy writes proceed now? | No. Block writes until `index_members` exact-dedup planning and `index_weights` PIT semantics decision are complete. | Authorize direct copy write anyway. | Direct copy would preserve 12,300 exact duplicate rows in `index_members` and blur the mixed PIT status in `index_weights`. |
| D-B1B-BLOCK-002 | implementation | How should `index_members` exact full-row duplicates be handled? | Run a read-only exact-dedup candidate planning gate, analogous to Batch 1A-exact-dedup, retaining one exact copy per full row. | Preserve duplicates as-is. | Preserving duplicates weakens CR139 write dedup guarantees and pollutes candidate truth. |
| D-B1B-BLOCK-003 | architecture | How should `index_weights` mixed `pit_status` be handled? | Preserve row-level `pit_status` and require a semantics note that `index_weights` is not a substitute for `index_members`; candidate write can only proceed after reviewer accepts mixed PIT status as expected evidence, not an error. | Treat any `pit_incomplete` row as blocking all writes. | Blocking may be too conservative for a candidate copy; silently treating the table as fully PIT available is unsafe. |
| D-B1B-BLOCK-004 | implementation | Should the write plan be updated before any Batch 1B write? | Yes. Add a Batch 1B-exact-dedup subcategory for `index_members` and a mixed-PIT semantics note for `index_weights`. | Leave write plan unchanged and rely on check files. | Future agents may miss the readiness findings if they only read the write plan. |
| D-B1B-BLOCK-005 | runtime_authorization | What remains explicitly not authorized? | Batch 1B write, Batch 2, catalog/provider/provider-lake-catalog writes, pointer advance, physical migration, credential read, NAS/runtime/QMT/MiniQMT/gateway/trading/live, Git remote write, rollback deletion. | Blanket-authorize after this blocker brief. | Blanket authorization would skip required evidence review and could mutate lake state prematurely. |

## Recommended Next Authorization Text

```text
authorize Gate B Batch 1B blocker planning only: index_members exact-dedup candidate planning + index_weights mixed PIT semantics decision, no lake write
```

## Exact Replies

- `approve` means: accept the recommendations above and authorize only the read-only Batch 1B blocker planning gate. It does not authorize Batch 1B write.
- `修改: <具体修改点>` means: change one or more decisions before planning.
- `reject` means: do not continue Batch 1B now.

## Explicit Non-Authorization

This brief does not authorize:

- Batch 1B lake write.
- Batch 2.
- Catalog/provider catalog/provider-lake-catalog write.
- Published pointer advance.
- Physical partition migration.
- Credential read.
- NAS/runtime/QMT/MiniQMT/gateway/trading/live.
- Git remote write.
- Rollback deletion.

