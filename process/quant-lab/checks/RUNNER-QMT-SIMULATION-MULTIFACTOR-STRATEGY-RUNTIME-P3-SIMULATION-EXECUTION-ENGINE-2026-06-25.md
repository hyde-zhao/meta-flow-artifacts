---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P3-SIMULATION-EXECUTION-ENGINE-2026-06-25"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T11:47:06+08:00"
owner: "host-orchestrator"
phase_goal: "P3-simulation-execution-engine"
stage_decision: "PASS"
target_status: "simulation-execution-engine-ready"
evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P3-EVIDENCE-2026-06-25.json"
runtime_authorization_granted: false
---

# P3 Simulation Execution Engine 检查结果

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `PASS` |
| 当前 P3 状态 | `simulation-execution-engine-ready` |
| 下一阶段 | `P4-reconciliation-and-order-report-verification` |
| Evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P3-EVIDENCE-2026-06-25.json` |

P3 已实现并验证模拟执行引擎合同：从已通过的 P2 order plan 进入 caller-provided simulation gateway interface，支持先撤单后提交、批量 submit/cancel 控制、runtime identity gate、异常停止和人工接管标记。本阶段只使用内存 fake gateway 测试，不启动真实 QMT runtime、不读取凭据、不执行真实 simulation/live。

## 2. Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| P2 order plan 已通过 | PASS | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P2-SIMULATION-ORDER-PLAN-2026-06-25.md` | P3 只消费 generated plan |
| Stage gate 合同可用 | PASS | `trading/stage_gate.py` | P3 调用 `evaluate_stage_gate` |
| P0 runtime identity 边界可用 | PASS | P0 check | P3 要求 expected runtime identity |
| 不授权真实 runtime | PASS | 用户指令 + evidence | 只使用 fake gateway |

## 3. Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | Execution engine 输入合同存在 | PASS | `SimulationExecutionEngineRequest` | 包含 order plan、stage gate、authorization ref、expected runtime identity |
| 2 | Runtime identity gate | PASS | 新增测试 | `expected_runtime_mode != simulation` 时 gateway 调用数为 0 |
| 3 | Stage gate 接入 | PASS | `execute_simulation_order_plan` | gate 不通过时 blocked |
| 4 | P2 blocked plan 不执行 | PASS | `execute_simulation_order_plan` | order plan blocked 时进入人工接管 |
| 5 | 先撤单后提交 | PASS | 新增测试 | fake cancel 先执行，再 submit |
| 6 | 批量控制 | PASS | `throttle_batch_size` | 超出 batch size 停止后续 submit |
| 7 | 异常停止 / 人工接管 | PASS | `manual_takeover_required` | blocked/rejected/unknown 均停止 |
| 8 | 脱敏输出 | PASS | `SimulationExecutionEngineResult.to_dict` | 不输出原始 symbol / raw broker order ref |

## 4. Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| P3 engine 可被 P4 对账消费 | PASS | P3 result schema | 输出 submitted/cancelled/rejected/unknown 计数 |
| 关键 fail-closed 覆盖 | PASS | 新增测试 | runtime profile mismatch 在 gateway 前阻断 |
| 验证通过 | PASS | `29 passed` + py_compile | P0-P3 相关测试通过 |

## 5. Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Execution engine 合同 | `trading/strategy_runner/simulation_activation.py` | PASS | 新增 P3 API，旧 activation API 保持兼容 |
| Order plan 合同 | `trading/strategy_runner/simulation_order_plan.py` | PASS | 为 P3 提供 passed intents |
| 包导出 | `trading/strategy_runner/__init__.py` | PASS | 导出 P3 API |
| 测试 | `tests/test_runner_multifactor_runtime_pipeline.py` | PASS | 覆盖 P1-P3 串联、风控阻断、runtime mismatch |
| Evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P3-EVIDENCE-2026-06-25.json` | PASS | 结构化证据 |

## 6. 验证命令

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr020_runtime_manual_validation.py tests/test_cr020_linux_client_rest_transport.py tests/test_runner_multifactor_runtime_pipeline.py tests/test_runner_simulation_activation.py` | PASS | `29 passed in 0.34s` |
| `PYTHONPYCACHEPREFIX=/tmp/p1-p3-runtime-pycompile uv run --python 3.11 python -m py_compile trading/strategy_runner/target_portfolio.py trading/strategy_runner/simulation_order_plan.py trading/strategy_runner/simulation_activation.py trading/strategy_runner/__init__.py tests/test_runner_multifactor_runtime_pipeline.py tests/test_runner_simulation_activation.py` | PASS | 无语法错误 |

## 7. 阶段决策

结论：`PASS`

P3 已完成。下一步建议进入 P4：对账与订单回报验证。P4 如需触达真实 QMT runtime、账户、订单回报或凭据，必须另行进行 runtime_authorization；本阶段未授予该权限。
