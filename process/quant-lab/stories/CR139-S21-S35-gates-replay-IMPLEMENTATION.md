---
change_id: "CR-139"
batch_id: "CR139-BACKLOG-A8"
stories: ["CR139-S21", "CR139-S35"]
stage: "CP6"
status: "complete"
implemented_at: "2026-06-30T15:55:00+08:00"
implemented_by: "host-orchestrator-inline"
validation_mode: "static-fixture"
---

# CR139 Backlog-A8 S21/S35 Implementation

## Scope

| Story | Implemented | Non-scope |
|---|---|---|
| S21 | Local cost pretrade gate contract in `trading/qmt_gateway_contracts.py`. | QMT runtime, broker adapter calls, account query, order submit. |
| S35 | Published as_of replay selector in `market_data/replay.py` and explicit-pointer CLI wrapper in `market_data/cli.py`. | Provider fetch, real lake scan, catalog/manifest/pointer writes, physical replay execution. |

## Implementation Objects

| File | Change |
|---|---|
| `trading/qmt_gateway_contracts.py` | Added S21 stable reason codes, `CostPretradeGateInput`, `CostPretradeGateDecision`, and `evaluate_cost_pretrade_gate()`. |
| `market_data/replay.py` | Added S35 stable reason codes, `PublishedAsOfReplayRequest`, `PublishedAsOfReplayResult`, and `build_published_asof_replay()`. |
| `market_data/cli.py` | Added `cmd_published_asof_replay()` and `published-asof-replay` parser entry accepting explicit pointer JSON. |
| `tests/test_cr139_commission_cost_pretrade_gate.py` | Added S21 fixture tests. |
| `tests/test_cr139_replay_published_asof.py` | Added S35 fixture tests. |

## Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| S21 must fail closed when schedule/version/cost/liquidity/tradability evidence is missing. | `evaluate_cost_pretrade_gate()` returns stable blocked reasons. | `tests/test_cr139_commission_cost_pretrade_gate.py` |
| S21 must keep all real operation counters at zero. | `CostPretradeGateDecision.operation_counters` uses QMT forbidden counter fields with zero values. | Counter assertion test. |
| S35 must replay only from exact published as_of pointer payloads. | `build_published_asof_replay()` matches dataset + `as_of_trade_date`, requires published status/path/manifest ref. | `tests/test_cr139_replay_published_asof.py` |
| S35 must not backfill from provider or scan real lake. | Function accepts explicit in-memory pointer records; CLI wrapper accepts explicit JSON payload only. | Missing as_of and CLI wrapper tests. |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_commission_cost_pretrade_gate.py tests/test_cr139_replay_published_asof.py tests/test_cr139_config_facts_versioning.py tests/test_cr138_gateway_query_calendar_commission_pnl.py tests/test_cr014_p0_pipeline_contract.py` | PASS, 28 passed in 0.59s |
| `uv run --python 3.11 python -m py_compile trading/qmt_gateway_contracts.py market_data/replay.py market_data/cli.py` | PASS |

## Boundary Check

No provider fetch, credential read, QMT/runtime/trading operation, real lake read/list/write, catalog/manifest write, published pointer advance, physical migration, NAS operation, or Git remote write was performed.
