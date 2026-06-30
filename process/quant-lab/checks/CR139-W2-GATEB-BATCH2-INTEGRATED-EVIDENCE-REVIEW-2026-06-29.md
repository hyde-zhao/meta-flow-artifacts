# CR139 W2 Gate B Batch 2 Integrated Evidence Review - 2026-06-29

## Decision

PASS_WITH_RISK_GATE_B_BATCH2_CANDIDATE_LAYER_COMPLETE

Gate B Batch 2 candidate evidence is accepted. All seven Batch 2 datasets have additive-only CR139 candidate objects written and verified. This review approves the candidate layer only. It does not authorize Gate C catalog/manifest writes, Gate D published pointer advance, Gate E physical migration, NAS/runtime/credential access, Git remote write, legacy deletion, or rollback deletion.

## Findings

No blocking findings.

| Severity | ID | Finding | Impact | Recommendation |
|---|---|---|---|---|
| LOW | GATEB-B2-REVIEW-001 | Zero-mutation proof canonical field names are not uniform across datasets. | Future machine consumers need schema-aware handling for `canonical_*_tree_hash_unchanged` fields. | Normalize proof schema before Gate C integrated automation, or keep review adapter aware of field variants. |
| LOW | GATEB-B2-REVIEW-002 | Completion ledger `operation_counts` are scoped to the final three large-table writes, while the integrated total is 14 candidate object writes plus 2 guarded old-candidate deletes. | Not a data defect; audit readers must use dataset evidence indexes for totals. | Use this integrated review as the authoritative Gate B Batch2 rollup. |

## Dataset Evidence

| Dataset | Source rows | Main rows | Quarantine rows | Dropped exact duplicates | Main duplicate over unique | Objects | Evidence |
|---|---:|---:|---:|---:|---:|---:|---|
| `liquidity_capacity` | 17,095,488 | 17,093,658 | 1,830 | 0 | 0 | 2 | `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-STRATEGY-UNIFICATION-EXECUTION.index.json` |
| `market_cap` | 17,108,103 | 17,001,327 | 106,776 | 0 | 0 | 2 | `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-STRATEGY-UNIFICATION-EXECUTION.index.json` |
| `adj_factor` | 29,970,830 | 17,905,960 | 321,815 | 11,743,055 | 0 | 2 | `process/evidence/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION.index.json` |
| `events` | 362,887 | 350,753 | 12,134 | 0 | 0 | 2 | `process/evidence/CR139-W2-GATEB-BATCH2-EVENTS-WRITE-EXECUTION.index.json` |
| `prices` | 28,646,730 | 17,013,700 | 401,628 | 11,231,402 | 0 | 2 | `process/evidence/CR139-W2-GATEB-BATCH2-PRICES-WRITE-EXECUTION.index.json` |
| `prices_limit` | 40,962,525 | 19,210,017 | 11,841,929 | 9,910,579 | 0 | 2 | `process/evidence/CR139-W2-GATEB-BATCH2-PRICES_LIMIT-WRITE-EXECUTION.index.json` |
| `trade_status` | 21,569,907 | 13,957,403 | 7,612,504 | 0 | 0 | 2 | `process/evidence/CR139-W2-GATEB-BATCH2-TRADE_STATUS-WRITE-EXECUTION.index.json` |

## Integrated Totals

| Metric | Value |
|---|---:|
| Datasets | 7 |
| Candidate objects | 14 |
| Source rows accounted | 155,716,470 |
| Main candidate rows | 102,532,818 |
| Quarantine rows | 20,298,616 |
| Dropped exact duplicate rows | 32,885,036 |

## Mechanical Validation

| Check | Result |
|---|---|
| Process route health | PASS |
| Manifest objects checked | 14/14 |
| Manifest objects match file existence, size, SHA-256, and parquet row count | 14/14 |
| Main candidates have duplicate-key over unique = 0 | 7/7 |
| Zero-mutation proofs pass | 6/6 execution groups |
| Candidate layer coverage | 7/7 Batch2 datasets |
| Event ledger check | PASS |
| JSON parse | PASS |
| `git diff --check` | PASS |

## Boundary

The following remain unauthorized and unexecuted:

- catalog write;
- provider catalog/provider-lake-catalog write;
- published pointer advance;
- physical partition migration;
- credential read;
- NAS operation;
- runtime/QMT/MiniQMT/gateway/trading operation;
- Git remote write;
- legacy deletion;
- rollback deletion.

## Residual Risks Before Gate C/D/E

- Gate C must review catalog/manifest shape and lineage checksum readiness before writing metadata.
- Gate D must separately review published pointer semantics; candidate rows are not current truth yet.
- Gate E physical migration remains separate and should stay after catalog/pointer readiness decisions.
- Quarantine objects preserve conflict data, but reader exposure must remain blocked until catalog/pointer policy is approved.

## Next Gate

Recommended next step: Gate C catalog/manifest write planning review only. Do not advance published pointer or perform physical migration in the same authorization.

Machine index: `process/evidence/CR139-W2-GATEB-BATCH2-INTEGRATED-EVIDENCE-REVIEW.index.json`


## Review Follow-up Cleanup

Follow-up evidence: `process/evidence/CR139-W2-GATEB-BATCH2-REVIEW-FOLLOWUP-CLEANUP-2026-06-29.json`.

| Item | Status | Detail |
|---|---|---|
| Empty old `liquidity_capacity` `...-canonical` candidate directory | CLOSED | Removed the empty directory shell with exact `rmdir`; no parquet/data objects were deleted. The resolved `liquidity_capacity` candidate remains present with 2 parquet files. |
| `events` quarantine count basis | CLOSED | `events` quarantine rows are full quarantined rows, not over-unique excess rows. The `12,134` quarantine rows consist of `10,583` metadata-superseded rows plus `1,551` rows from `774` business-conflict groups. This is intentionally greater than the historical over-unique count (`11,360`). |
| Dropped exact duplicate row split | ALREADY SATISFIED | The Dataset Evidence table already lists dropped values per dataset: `adj_factor=11,743,055`, `prices=11,231,402`, `prices_limit=9,910,579`, total `32,885,036`. |
| Batch2 rollup ledger | CLOSED | Added an authoritative rollup ledger event with candidate object total `14`, source rows `155,716,470`, main rows `102,532,818`, quarantine rows `20,298,616`, dropped rows `32,885,036`, and guarded old-candidate object deletes `2`. |
