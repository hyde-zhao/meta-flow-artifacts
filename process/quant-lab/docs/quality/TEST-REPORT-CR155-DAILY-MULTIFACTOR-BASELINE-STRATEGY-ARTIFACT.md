# CR155 Test Report

## Results

| Command | Result |
|---|---|
| `uv run pytest -q tests/research/test_daily_multifactor_baseline_artifact.py` | PASS: 11 passed |
| `uv run pytest -q tests/research/test_strategy_admission_statistical_gate.py tests/research/test_cross_strategy_reliability_gates.py tests/research/test_strategy_admission_package.py` | PASS: 24 passed |
| `uv run pytest -q tests/research/test_daily_multifactor_baseline_artifact.py tests/research/test_strategy_admission_statistical_gate.py tests/research/test_cross_strategy_reliability_gates.py tests/research/test_strategy_admission_package.py` | PASS: 35 passed |
| `uv run --python 3.11 python scripts/research/run_multifactor_strategy_research.py --run-id cr155-real-lake-baseline-rerun-a ...` | Completed; admission blocked |
| `uv run --python 3.11 python scripts/research/run_multifactor_strategy_research.py --run-id cr155-real-lake-baseline-rerun-b ...` | Completed; admission blocked with identical core metrics |

## Coverage

| Area | Covered Tests / Evidence | Result |
|---|---|---|
| Artifact required fields and deterministic JSON-safe serialization | S01 tests | PASS |
| Overclaim denial | S01 and S05 wording tests | PASS |
| Readonly provenance counters and fixture/static fallback | S02 tests plus real lake forbidden counter evidence | PASS |
| Walk-forward split and validation metrics | S03 tests plus real factor model validation report | PASS_WITH_RISK |
| Admission package status and `paper_candidate` derivation | S04 tests plus real admission package | PASS_WITH_RISK: `paper_candidate=false` |
| Rerun metric/status drift | S05 tests plus two real lake reruns | PASS |
| CR151/CR154/package compatibility | Related regression suite | PASS |

## Real Lake Result

| Metric | Rerun A | Rerun B | Match |
|---|---:|---:|---|
| total_return | -0.5889592112873733 | -0.5889592112873733 | yes |
| max_drawdown | -0.9695418716497479 | -0.9695418716497479 | yes |
| turnover | 0.336568760539169 | 0.336568760539169 | yes |
| cost | 5.048531408087536 | 5.048531408087536 | yes |
| admission status | BLOCKED | BLOCKED | yes |

## Gaps

| Gap | Status | Reason |
|---|---|---|
| Real lake historical/OOS/rerun execution | executed | Evidence in `process/evidence/CR155-REAL-LAKE-VALIDATION-20260704.json`. |
| Runtime/paper/live/trading/broker validation | not run | Not authorized and explicitly out of scope. |
| Strategy remediation to pass admission | not run | Current CR validates baseline artifact; remediation requires CP8 decision or follow-up CR. |
