---
cr_id: CR-158
stage: CP7
status: PASS
created_at: 2026-07-05T18:32:00+08:00
---

# CR158 Test Report

| Suite | Command | Result | Summary |
|---|---|---|---|
| CR158 targeted adapter contracts | `uv run pytest tests/research/test_strategy_type_adapter_core.py tests/research/test_event_strategy_adapter.py tests/research/test_ml_strategy_adapter.py tests/research/test_strategy_adapter_evidence_refs.py tests/research/test_strategy_adapter_no_runtime_guard.py` | PASS | 20 passed in 0.04s |
| Syntax | `uv run --python 3.11 python -m py_compile engine/strategy_type_adapters.py` | PASS | Module compiles |
| Related regression | `uv run pytest tests/research/test_event_driven_strategy_e2e_contracts.py tests/research/test_ml_strategy_e2e_contracts.py tests/research/test_strategy_admission_package.py tests/research/test_cross_strategy_reliability_gates.py` | PASS | 60 passed in 0.60s |
| Static diff | `git diff --check -- engine/strategy_type_adapters.py tests/research/test_*strategy_adapter*.py tests/research/test_strategy_type_adapter_core.py process/stories/CR158-EVENT-ML-STRATEGY-ADAPTER-IMPLEMENTATION.md` | PASS | No whitespace errors |

## Coverage Summary

| Story | Test Coverage | Result |
|---|---|---|
| CR158-S01 | Shared required groups, private-field denylist, nonzero counter blocked | PASS |
| CR158-S02 | Event P0 refs, missing alignment policy, live listener counter, ML leakage | PASS |
| CR158-S03 | ML P0 refs, missing validation report, registry counter, event leakage | PASS |
| CR158-S04 | Evidence refs generation, body copy blocking, public handoff surface | PASS |
| CR158-S05 | Zero counters, feed counter, registry counter, counter family coverage | PASS |
| CR158-S06 | Boundary validated through limitations, quality report and CP8 input | PASS_WITH_RISK |

## Not Run

| Scope | Reason |
|---|---|
| Real feed/training/provider/lake/NAS/credential/QMT/runtime/simulation/paper/live/trading/broker/store/registry/publish/external framework/Git remote | Not authorized by CP5/CP6/CP7; outside CR158 local/static/fixture scope |
