---
check_id: "RUNNER-MODEL-SIMULATION-LIVE-ENTRY-VERIFICATION-2026-06-24"
status: "PASS"
created_at: "2026-06-24T00:00:00+08:00"
owner: "host-orchestrator"
---

# Runner Model / Simulation / Live Entry Verification

## 1. 验证范围

本次验证覆盖新增 simulation 真实操作入口与旧安全门回归。

## 2. 自动化结果

| 命令 | 结果 |
|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr020_linux_client_rest_transport.py tests/test_runner_simulation_activation.py tests/test_cr020_runtime_manual_validation.py tests/test_cr128_runner_core_mvp.py` | PASS，30 passed |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr016_simulation_order_enable_gate.py tests/test_cr015_pretrade_risk_gate.py tests/test_cr015_oms_state_machine.py tests/test_cr019_qmt_endpoint_matrix.py tests/test_cr019_qmt_cside_client_cli.py tests/test_cr019_qmt_gateway_run_gates.py` | PASS，62 passed |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr020_linux_client_rest_transport.py tests/test_runner_simulation_activation.py tests/test_cr020_runtime_manual_validation.py tests/test_cr128_runner_core_mvp.py tests/test_cr016_simulation_order_enable_gate.py tests/test_cr015_pretrade_risk_gate.py tests/test_cr015_oms_state_machine.py tests/test_cr019_qmt_endpoint_matrix.py tests/test_cr019_qmt_cside_client_cli.py tests/test_cr019_qmt_gateway_run_gates.py tests/test_cr132_process_artifact_hygiene.py` | PASS，109 passed |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_process_artifact_hygiene.py --json` | PASS，`unclassified=0`，`allowed_to_enter_runner_development=true` |
| `PYTHONPYCACHEPREFIX=/tmp/runner-simulation-pycompile uv run --python 3.11 python -m py_compile trading/qmt_client.py trading/strategy_runner/simulation_activation.py trading/strategy_runner/__init__.py trading/qmt_gateway_contracts.py trading/qmt_runtime.py tests/test_cr020_linux_client_rest_transport.py tests/test_runner_simulation_activation.py tests/test_cr020_runtime_manual_validation.py` | PASS |
| `git diff --check` | PASS |

## 3. 阶段结论

结论：`PASS`

本地代码已经具备从模型盘目标组合进入模拟账户真实操作入口的受控实现切片。实盘资金阶段仍需独立授权门。
