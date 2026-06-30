---
story_id: "CR139-S25"
story_slug: "decision-time-lookahead-gate"
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-30T13:35:00+08:00"
implemented_by: "host-orchestrator-inline"
context_ref: "process/context/stories/STORY-CR139-S25.CP6.work-packet.json"
---

# CR139-S25 Decision Time Lookahead Gate Implementation

## Implementation Scope

S25 adds an explicit `decision_time` keyword to `read_dataset`. When supplied, the reader validates that the filtered frame contains an `available_at` column and every participating row satisfies `available_at <= decision_time`.

Default behavior remains unchanged when `decision_time` is omitted.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Pass rows available before decision time | `_decision_time_lookahead_gate()` returns `None` | `test_s25_decision_time_gate_allows_available_rows` |
| Block future `available_at` | `decision_time_lookahead_blocked` issue | `test_s25_decision_time_gate_blocks_future_available_at` |
| Require explicit `available_at` | `decision_time_available_at_missing` issue | `test_s25_decision_time_gate_requires_available_at_column` |
| Reject invalid decision time | `decision_time_unparseable` issue | `test_s25_decision_time_gate_blocks_invalid_decision_time` |
| Preserve S28/read path behavior | No S28 helper changes; regression tests | S28/PIT/dedup tests |

## Changed Files

| File | Change |
|---|---|
| `market_data/readers.py` | Added optional `decision_time` keyword and pure lookahead gate helper. |
| `tests/test_cr139_decision_time_lookahead_gate.py` | Added S25 fixture/static tests. |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_decision_time_lookahead_gate.py tests/test_cr139_readiness_pre_read_gate.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_read_layer_dedup.py` | PASS, 18 passed in 0.64s |

## Boundary Check

Fixture/static only. Test file writes occur under pytest `tmp_path`; no production lake/catalog/manifest/pointer/NAS/provider/runtime/Git operation was performed.
