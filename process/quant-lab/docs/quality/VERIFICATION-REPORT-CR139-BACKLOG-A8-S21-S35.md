---
change_id: "CR-139"
batch_id: "CR139-BACKLOG-A8"
stories: ["CR139-S21", "CR139-S35"]
stage: "CP7"
status: "PASS"
verified_at: "2026-06-30T16:05:00+08:00"
verified_by: "host-orchestrator-inline"
validation_mode: "static-fixture"
---

# CR139 Backlog-A8 S21/S35 Verification Report

## Scope

| Story | Scope | Non-scope |
|---|---|---|
| S21 | Local commission/cost/tradability pretrade gate contract. | QMT runtime, broker adapter call, account query, order submit, trading execution. |
| S35 | Published as_of replay selector and explicit-pointer CLI wrapper. | Provider fetch, real lake scan, catalog/manifest/pointer writes, physical replay execution. |

## Traceability

| Requirement | Story | Implementation | Verification | Result |
|---|---|---|---|---|
| REQ-241 | S21 | `CostPretradeGateInput`, `CostPretradeGateDecision`, `evaluate_cost_pretrade_gate()` | `tests/test_cr139_commission_cost_pretrade_gate.py` | PASS |
| REQ-245 | S35 | `PublishedAsOfReplayRequest`, `PublishedAsOfReplayResult`, `build_published_asof_replay()`, CLI wrapper | `tests/test_cr139_replay_published_asof.py` | PASS |
| S18/S04/S14 regressions | S21/S35 dependencies | Existing schedule and replay contracts remain compatible | config, gateway, and P0 replay regression tests | PASS |

## Commands

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_commission_cost_pretrade_gate.py tests/test_cr139_replay_published_asof.py tests/test_cr139_config_facts_versioning.py tests/test_cr138_gateway_query_calendar_commission_pnl.py tests/test_cr014_p0_pipeline_contract.py` | PASS, 28 passed in 0.59s |
| `uv run --python 3.11 python -m py_compile trading/qmt_gateway_contracts.py market_data/replay.py market_data/cli.py` | PASS |
| `uv run meta-flow story return-check --packet process/context/stories/STORY-CR139-S21.CP6.work-packet.json --return process/returns/STORY-CR139-S21.CP6.return.json` | PASS |
| `uv run meta-flow story return-check --packet process/context/stories/STORY-CR139-S35.CP6.work-packet.json --return process/returns/STORY-CR139-S35.CP6.return.json` | PASS |

## Boundary Verification

No QMT/runtime/trading operation, broker/account query, credential read, provider fetch, real lake read/list/write, active catalog/manifest write, published pointer advance, physical partition migration, NAS operation, or Git remote write was performed.

## Findings And Risks

No CP7 findings.

Remaining production operations, if ever needed, are out of this Story verification scope and require separate authorization: broker/runtime integration, real current pointer replay, physical replay execution, provider fetch, catalog/manifest publication, pointer movement, NAS sync, and trading execution.

## Stage Decision

Decision: `PASS`. S21 and S35 can move to `verified`.
