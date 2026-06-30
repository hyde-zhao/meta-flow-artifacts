---
story_id: "CR139-S30"
story_slug: "runid-naming-convention"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-29T12:00:00+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S30.CP6.work-packet.json"
---

# CR139-S30 Run ID Naming Convention Implementation

## Implementation Scope

S30 adds deterministic W2 run_id construction and static legacy `run_id=` path detection in `market_data/lake_layout.py`, plus normalization-facing wrappers in `market_data/normalization.py`.

The implementation does not migrate physical partitions and does not write lake data.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Deterministic W2 run_id | `build_cr139_run_id()` / `build_canonical_run_id()` | `test_cr139_run_id_is_deterministic_and_valid` |
| Naming validation | `validate_cr139_run_id()` | `tests/test_cr139_runid_naming.py` |
| Legacy path audit-only detection | `legacy_run_id_path_detected()` and `validate_partition_naming_contract()` | `test_legacy_run_id_partition_path_is_blocked_by_static_contract` |
| No migration execution | result includes `physical_migration_executed=False`, `lake_write_count=0` | `tests/test_cr139_runid_naming.py` |

## Changed Files

| File | Change |
|---|---|
| `market_data/lake_layout.py` | Added CR139 W2 run_id pattern, builder, validator, and legacy path detector |
| `market_data/normalization.py` | Added normalization-facing static naming contract helpers |
| `tests/test_cr139_runid_naming.py` | Added S30 fixture/static tests |

## Verification

| Command | Result |
|---|---|
| W2 + CR139 fixture/static regression | PASS, 88 passed in 3.07s |

## Boundary Check

Physical migration, real lake writes, provider operations, credential reads, and pointer advance execution were not executed.
