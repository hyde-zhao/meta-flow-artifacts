# CR139 W2 Gate B Batch 2 Remaining Split Planning - 2026-06-29

## Decision

PASS_WITH_RISK_SPLIT_PLANNING_COMPLETE_WRITE_DECISIONS_REQUIRED

The authorized read-only planning gate completed for `adj_factor`, `prices`, `prices_limit`, `events`, and `trade_status`. It did not write lake objects, delete candidates, write catalog/provider catalog/provider-lake-catalog, advance published pointers, migrate physical partitions, read credentials, touch NAS/runtime/trading, or use Git remote.

## Scope

Authorized scope: authorize Gate B Batch 2 remaining datasets consolidated read-only split planning only for adj_factor, prices, prices_limit, events, trade_status; no lake write

Evidence:

- Detail: `process/evidence/CR139-W2-GATEB-BATCH2-REMAINING-SPLIT-PLANNING-2026-06-29.json`
- Index: `process/evidence/CR139-W2-GATEB-BATCH2-REMAINING-SPLIT-PLANNING.index.json`
- Updated write plan JSON: `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.json`
- Updated write plan MD: `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.md`

## Machine Validation

| Check | Result |
|---|---|
| Detail JSON parses | PASS |
| Index JSON parses | PASS |
| Dataset count is 5 | PASS |
| `operation_counts` all zero | PASS |
| Candidate target dirs for the 5 remaining datasets absent before write | PASS |
| Split bucket totals match dataset sums | PASS |
| Lake write remains unauthorized | PASS |

## Split Results

| Dataset | Source rows | Exact-copy groups | Metadata-only groups | Business-conflict groups | Planned main rows | Planned quarantine rows | Planned exact-copy drops |
|---|---:|---:|---:|---:|---:|---:|---:|
| `adj_factor` | 29,970,830 | 11,743,055 | 81,852 | 0 | 17,905,960 | 321,815 | 11,743,055 |
| `prices` | 28,646,730 | 11,231,402 | 1,830 | 79,958 | 17,013,700 | 401,628 | 11,231,402 |
| `prices_limit` | 40,962,525 | 9,910,579 | 6,946,008 | 499,787 | 19,210,017 | 11,841,929 | 9,910,579 |
| `events` | 362,887 | 0 | 10,583 | 774 | 350,753 | 12,134 | 0 |
| `trade_status` | 21,569,907 | 0 | 70,087 | 3,692,105 | 13,957,403 | 7,612,504 | 0 |

Totals:

- Source rows: 121,512,879
- Duplicate-key groups: 44,268,020
- Duplicate rows in groups: 93,070,442
- Exact-copy groups: 32,885,036
- Metadata-only groups: 7,110,360
- Business-conflict groups: 4,272,624
- Planned main rows if recommended actions are approved: 68,437,833
- Planned quarantine rows if recommended actions are approved: 20,190,010
- Planned exact-copy dropped rows: 32,885,036

## Important Findings

1. `adj_factor` has no business-conflict groups. It is the cleanest remaining Batch 2 write candidate, but still requires explicit approval for exact-copy dedup and metadata source precedence before any lake write.
2. `prices` has 79,958 business-conflict groups and schema variants. It requires a prices schema-normalization decision before write.
3. `prices_limit` has 499,787 business-conflict groups and a large metadata-only bucket. It needs a per-dataset decision; do not batch-write it with cleaner tables.
4. `events` now splits into 10,583 metadata-only groups and 774 business-conflict groups. The older single number `351,527` usable rows is superseded by the split-planning candidate shape: 350,753 main rows plus 12,134 quarantine rows if the recommended split is approved.
5. `trade_status` is mostly business-conflict groups: 3,692,105 groups / 7,542,079 rows. It should remain a separate single-dataset authorization.

## Implementation Note

An initial implementation attempted full source-run set enumeration for every duplicate-key group and was interrupted before any output file was written because the aggregation cost was too high for the 100M+ row planning scope. The final planner keeps full-table bucket counts and limited samples, which is sufficient for routing and authorization decisions while preserving the no-write boundary.

## Next Recommendation

Recommended next write candidate: `adj_factor` only, after a separate authorization explicitly approves:

- exact-copy dedup drop policy;
- metadata-only source precedence for 81,852 groups;
- additive-only candidate write to the new CR139 run-id path;
- no catalog, pointer, physical migration, NAS/runtime/credential/Git remote, legacy deletion, or rollback.

`prices`, `prices_limit`, `events`, and `trade_status` require per-dataset decisions before write authorization.
