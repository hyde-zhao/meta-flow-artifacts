# CR139 W2 Gate B Batch 1A Remaining Readiness Decision Brief

Generated at: 2026-06-29T15:06:25+08:00

## Approver Summary

Batch 1A remaining consolidated pre-write readiness was executed in read-only mode. It did not write lake objects, catalog, pointers, physical partitions, credentials, NAS, runtime, or Git remote.

The consolidated readiness check is **not ready for all 7 datasets** because `trade_calendar` and `stock_basic` contain exact full-row duplicates under the Gate A inventory source scope (`physical_canonical_root`). The other 5 datasets are ready for a narrow additive-only candidate copy write.

Recommended decision: approve only the 5 ready datasets for the next write gate, and defer `trade_calendar` / `stock_basic` until an exact-dedup candidate strategy is explicitly approved.

## Evidence Summary

| Item | Result |
|---|---|
| Gate A rerun drift count | 0 |
| Path collision count | 0 |
| Target roots existing before write | 0 |
| Unauthorized operation counts | all 0 |
| Total profiled rows | 181075 |
| Total source files | 56 |
| Full-row duplicate over unique total | 11981 |

Evidence refs:

- Readiness index: `process/evidence/CR139-W2-GATEB-BATCH1A-REMAINING-PREWRITE-READINESS.index.json`
- Readiness detail: `process/evidence/CR139-W2-GATEB-BATCH1A-REMAINING-PREWRITE-READINESS-PHYSICAL-ROOT-2026-06-29T150625+0800.json`
- Readiness check: `process/checks/CR139-W2-GATEB-BATCH1A-REMAINING-PREWRITE-READINESS-2026-06-29.md`

## Decision Collection Coverage

| Source scanned | Candidate issue groups | Included decision items | N/A |
|---|---:|---:|---:|
| Gate A rerun + drift check | 1 | 1 | 0 |
| Full-row profiles | 2 | 2 | 0 |
| Path collision matrix | 1 | 1 | 0 |
| Authorization boundary | 1 | 1 | 0 |

All candidate issue groups were included below. Duplicate source-scope mismatch from the first local probe is recorded in the readiness check as corrected evidence, not as a pending decision.

## Pending Human Decisions

| ID | Type | Question | Recommended | Alternative | Risk / Impact |
|---|---|---|---|---|---|
| D-B1A-001 | runtime_authorization | Should the next write gate include all 7 remaining Batch 1A datasets? | No. Authorize only the 5 ready datasets: `lifecycle_code_change, hs300_index, industry_classification, prices_limit_code_change_fixes, prices_limit_coverage_exclusions`. | Authorize no writes and revise plan first. | Authorizing all 7 would copy exact full-row duplicates in `trade_calendar` and `stock_basic`. |
| D-B1A-002 | implementation | How should `trade_calendar` exact duplicates be handled? | Defer from copy-write; prepare exact-dedup candidate plan and evidence. | Preserve duplicates as-is. | Preserving exact duplicates weakens CR139 write dedup guarantee. |
| D-B1A-003 | implementation | How should `stock_basic` exact duplicates be handled? | Defer from copy-write; prepare exact-dedup candidate plan and evidence. | Preserve duplicates as-is. | `stock_basic` has 11,608 exact duplicate rows over unique; copying as-is is not a clean contract write. |
| D-B1A-004 | runtime_authorization | Should safe-5 candidate write use batch atomicity policy? | Yes: fail-stop, no automatic rollback, manifest partial completion, rollback only by separate authorization. | Auto-rollback on failure. | Auto-rollback would be a lake deletion operation and should not be implicit. |

## Exact Replies

- `approve` means: accept the recommendations above and authorize the next gate only for the 5 ready datasets additive-only candidate writes. It does not authorize `trade_calendar`, `stock_basic`, catalog, pointer, migration, NAS, runtime, credential, Git remote, or rollback deletion.
- `修改: <具体修改点>` means: change one or more decisions before the next gate.
- `reject` means: do not proceed to the safe-5 write gate.

## Recommended Next Authorization Text

`authorize Gate B Batch 1A safe 5 datasets additive-only candidate writes only: lifecycle_code_change, hs300_index, industry_classification, prices_limit_code_change_fixes, prices_limit_coverage_exclusions`

Still not authorized: `trade_calendar`, `stock_basic`, Batch 1B/2, catalog/provider catalog/provider-lake-catalog write, published pointer advance, physical partition migration, credential read, NAS/runtime/QMT/MiniQMT/gateway/trading/live, Git remote write, rollback deletion.
