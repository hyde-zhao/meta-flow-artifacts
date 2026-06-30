---
story_id: "CR139-S29"
story_slug: "pit-dedup-correctness-tests"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-30T11:05:00+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S29.CP6.work-packet.json"
---

# CR139-S29 PIT/Dedup Correctness Tests Implementation

## Implementation Scope

S29 adds fixture/static correctness tests for the S29 correctness standard:

- PIT future financial-report rows are invisible at `as_of`.
- The latest visible PIT row wins for a natural key.
- Read-layer duplicate natural keys are reduced to zero.
- Duplicate resolution is deterministic and prefers the catalog current run.

The implementation only adds tests and uses in-memory / `tmp_path` fixtures. It does not write the real lake, catalog, manifest, pointer, NAS, provider surfaces, runtime, or Git remote.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| `STD-S29-PIT-01` future rows invisible | `tests/test_cr139_pit_correctness.py` builds a future financial-report row and reads as-of before its availability | `test_s29_pit_standard_excludes_future_financial_report_rows` |
| `STD-S29-PIT-02` latest visible row wins | Same fixture contains two visible rows; the later visible row is selected | Same test |
| `STD-S29-DEDUP-01` zero duplicate natural keys | `tests/test_cr139_dedup_correctness.py` reads duplicate run partitions and counts duplicate `(symbol, trade_date)` groups | `test_s29_dedup_standard_returns_zero_duplicate_natural_keys` |
| `STD-S29-DEDUP-02` deterministic duplicate resolution | Fixture catalog marks `run-cr139-s29-current` as current; output retains that run | Same test |
| Boundary standard | Fixtures only; no production data mutation | Targeted pytest + CP6 boundary check |

## Changed Files

| File | Change |
|---|---|
| `tests/test_cr139_pit_correctness.py` | Added S29 PIT correctness fixture test. |
| `tests/test_cr139_dedup_correctness.py` | Added S29 read-layer dedup correctness fixture test. |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_pit_correctness.py tests/test_cr139_dedup_correctness.py tests/test_cr139_adjustment_factor_pit.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_read_layer_dedup.py` | PASS, 15 passed in 0.38s |

## Boundary Check

No real lake write, active catalog write, active manifest write, pointer advance, NAS operation, provider fetch, credential read, runtime/trading operation, physical partition migration, or Git remote write was performed.
