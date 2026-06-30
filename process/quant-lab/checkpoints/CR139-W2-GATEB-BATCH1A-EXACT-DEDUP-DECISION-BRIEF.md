# CR139 W2 Gate B Batch 1A Exact-Dedup Decision Brief

Generated at: 2026-06-29T15:25:49+08:00

## Approver Summary

`trade_calendar` and `stock_basic` were moved to `Batch 1A-exact-dedup` because direct copy-write would preserve exact full-row duplicates. This planning gate is read-only and recommends a deterministic exact-dedup candidate write for these two datasets only.

## Evidence Summary

| Dataset | Source Rows | Retained Rows | Dropped Exact Duplicate Rows | Candidate Shape |
|---|---:|---:|---:|---|
| `trade_calendar` | 10952 | 10579 | 373 | one parquet under `candidate/parquet/dataset=trade_calendar/schema_version=1.0/run_id=cr139-w2-trade_calendar-legacy_lake-20260629-canonical` |
| `stock_basic` | 55166 | 43558 | 11608 | one parquet under `candidate/parquet/dataset=stock_basic/schema_version=1.0/run_id=cr139-w2-stock_basic-legacy_lake-20260629-canonical` |


Retain rule: sort by `_cr139_source_file ASC`, `_cr139_source_row_number ASC`, then drop exact full-row duplicates keeping first. No quarantine is recommended because dropped rows are exact copies of retained rows.

## Pending Human Decisions

| ID | Type | Question | Recommended | Alternative | Risk / Impact |
|---|---|---|---|---|---|
| D-B1A-DEDUP-001 | implementation | Should `trade_calendar` use deterministic exact-dedup candidate write? | Yes, retain 10579 and drop 373 exact duplicate rows. | Preserve duplicates as-is. | Preserving duplicates weakens write dedup guarantee. |
| D-B1A-DEDUP-002 | implementation | Should `stock_basic` use deterministic exact-dedup candidate write? | Yes, retain 43558 and drop 11608 exact duplicate rows. | Preserve duplicates as-is. | Preserving 11,608 duplicate rows would pollute candidate truth. |
| D-B1A-DEDUP-003 | runtime_authorization | Should the next write gate be limited to these two dedup candidates? | Yes, authorize only `trade_calendar` and `stock_basic` dedup candidate writes. | Defer writes and review plan further. | Keeps write scope narrow and auditable. |

## Exact Replies

- `approve` means: accept the recommendations and authorize only the future exact-dedup write gate for `trade_calendar` and `stock_basic`.
- `修改: <具体修改点>` means: change one or more decisions before write authorization.
- `reject` means: do not proceed to exact-dedup writes.

Recommended authorization text:

```text
authorize Gate B Batch 1A-exact-dedup trade_calendar and stock_basic additive-only dedup candidate writes only
```

Still not authorized: catalog/provider-lake-catalog write, published pointer advance, physical migration, credential read, NAS/runtime/QMT/MiniQMT/gateway/trading/live, Git remote, rollback deletion, Batch 1B/2.
