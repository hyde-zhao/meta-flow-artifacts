---
story_id: "CR139-S32"
story_slug: "write-dedup-guarantee"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-29T12:00:00+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S32.CP6.work-packet.json"
---

# CR139-S32 Write Dedup Guarantee Implementation

## Implementation Scope

S32 adds a canonical write-time dedup validation helper in `market_data/normalization.py`. It fails closed by default and supports deterministic dedup only when explicitly requested.

The helper does not write parquet or mutate lake state.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Fail closed on duplicate primary key | `validate_canonical_write_dedup(..., policy="fail_on_duplicate")` raises `CanonicalDeduplicationError` | `test_write_dedup_fail_closed_on_duplicate_key` |
| Explicit deterministic dedup | `policy="deduplicate_deterministic"` sorts by key/available_at/source_run_id/checksum and keeps last | `test_write_dedup_can_be_deterministic_when_policy_explicit` |
| Evidence shape | `WriteDeduplicationResult.to_dict()` reports duplicate/dropped counts | W2 regression |

## Changed Files

| File | Change |
|---|---|
| `market_data/normalization.py` | Added `CanonicalDeduplicationError`, `WriteDeduplicationResult`, and `validate_canonical_write_dedup()` |
| `tests/test_cr139_write_dedup_guarantee.py` | Added S32 fixture/static tests |

## Verification

| Command | Result |
|---|---|
| W2 + CR139 fixture/static regression | PASS, 88 passed in 3.07s |

## Boundary Check

No real lake write, provider fetch, credential read, pointer advance execution, or physical migration was executed.
