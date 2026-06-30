---
story_id: "CR139-S31"
story_slug: "events-schema-repair"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-29T12:00:00+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S31.CP6.work-packet.json"
---

# CR139-S31 Events Schema Repair Implementation

## Implementation Scope

S31 adds in-memory canonical events schema repair in `market_data/normalization.py`. It fills explicit event `available_at`, `available_at_rule`, payload, source lineage, schema version, and checksum fields before write-time dedup validation.

No raw/provider fetch, real lake write, or publish step was executed.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Canonical events column set | `repair_events_schema()` returns `CANONICAL_EVENTS_COLUMNS` order | `test_events_schema_repair_fills_available_at_and_lineage_fields` |
| Available-at repair | missing `available_at` derives `09:20:00+08:00` from event_date | `tests/test_cr139_events_schema_repair.py` |
| Lineage fields | source, source_interface, source_run_id, schema_version, checksum defaulted explicitly | `tests/test_cr139_events_schema_repair.py` |
| Duplicate guard | repair calls `validate_canonical_write_dedup()` fail-closed | W2 regression |

## Changed Files

| File | Change |
|---|---|
| `market_data/normalization.py` | Added `EventsSchemaRepairResult` and `repair_events_schema()` |
| `tests/test_cr139_events_schema_repair.py` | Added S31 fixture/static test |

## Verification

| Command | Result |
|---|---|
| W2 + CR139 fixture/static regression | PASS, 88 passed in 3.07s |

## Boundary Check

All repair work is in-memory. Real lake writes, provider catalog writes, credential reads, and runtime operations were not executed.
