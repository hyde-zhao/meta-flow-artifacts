---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-REAL-OPERATOR-2026-06-25"
task_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T13:55:14+08:00"
owner: "host-orchestrator"
stage_decision: "BLOCKED_EXTERNAL_RUNTIME_SESSION"
runtime_authorization_ref: "runner-qmt-multifactor-runtime-20260625-simulation"
evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-REAL-OPERATOR-EVIDENCE-2026-06-25.json"
readonly_evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-REAL-READONLY-EVIDENCE-2026-06-25.json"
---

# Runner QMT Simulation Multifactor Real Operator Check

## 1. 结论

| 项目 | 结果 |
|---|---|
| 阶段决策 | `BLOCKED_EXTERNAL_RUNTIME_SESSION` |
| simulation runtime 授权 | 已由用户本轮请求授权，仅限 `simulation` |
| small_live / live | `NOT_AUTHORIZED` |
| gateway health | `PASS`，`session_ready=true`，`runtime_status=xtquant-ready` |
| signed positions readonly | `BLOCKED`，`reason_code=session_expired` |
| 多因子真实 operator | `BLOCKED`，阻断在 pre-positions 阶段 |
| simulation submit/cancel | 未执行 |
| P1-P4 真实链路 | 未进入，因 pre-positions session gate fail closed |
| 证据持久化 | 已写入 redacted summary-only evidence |

本轮已补齐真实 simulation operator 工程入口、证据持久化策略和稳定性窗口记录能力，并执行真实 gateway 探测。gateway 进程仍可达，但 signed POST 进入 session gate 后返回 `session_expired`；因此 operator 在读取 pre-positions 脱敏摘要前 fail closed，未生成真实 order plan、未提交订单、未撤单、未执行对账。

## 2. 已完成工作

| 工作项 | 状态 | 证据 |
|---|---|---|
| 显式 simulation runtime 授权边界 | 完成 | 本检查记录 `runtime_authorization_ref`；`small_live_or_live_authorized=false` |
| 只读持仓快照接入 | 阻断于 session gate | `REAL-READONLY-EVIDENCE`，原因 `session_expired` |
| P1-P4 真实 operator 入口 | 完成工程实现，真实执行阻断 | `trading/strategy_runner/simulation_operator.py` |
| 真实 simulation submit/cancel | 未执行 | pre-positions 阻断，submit/cancel 未触发 |
| 订单 / 持仓回执校验 | 未执行 | 无 submit/cancel 回执 |
| 稳定性窗口 | 已实现记录；当前 `collecting` | operator evidence `stability_window.completed_runs=0` |
| operator / 调度入口 | 完成 | `scripts/run_qmt_multifactor_simulation_operator.py` |
| 证据持久化策略 | 完成 | raw payload/account/symbol/broker ref 均 forbidden，retention_days=30 |

## 3. 验证命令

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_runner_multifactor_runtime_pipeline.py tests/test_runner_multifactor_ops_pipeline.py tests/test_runner_multifactor_simulation_operator.py tests/test_cr020_runtime_manual_validation.py tests/test_cr020_linux_client_rest_transport.py` | PASS | `33 passed` |
| `PYTHONPYCACHEPREFIX=/tmp/runner-qmt-multifactor-operator-pycompile uv run --python 3.11 python -m py_compile trading/strategy_runner/simulation_operator.py scripts/run_qmt_multifactor_simulation_operator.py tests/test_runner_multifactor_simulation_operator.py trading/strategy_runner/__init__.py` | PASS | 无语法错误输出 |
| `git diff --check` | PASS | 无空白错误 |
| `curl -sS --max-time 5 http://172.30.32.1:18765/qmt/health` | PASS | gateway reachable，health redacted |
| `scripts/collect_cr099_runner_readonly_smoke.py ...` | BLOCKED | evidence contract PASS，但 runtime result `session_expired` |
| `scripts/run_qmt_multifactor_simulation_operator.py ...` | BLOCKED | operator evidence 写入，阻断原因 `session_expired` |

## 4. 阻断条件

| ID | 状态 | 说明 | 下一步 |
|---|---|---|---|
| B-RUNTIME-SESSION-EXPIRED | OPEN | gateway 可达，但 signed positions readonly 返回 `session_expired`，说明真实 POST runtime session 已过期。 | 需要在 Windows QMT 节点重启 / 重新登录 gateway，使 session TTL 刷新后复跑 operator。 |

## 5. 安全边界

- 未执行 small_live / live。
- 未保存凭据、token、secret、账号原文、证券代码原文、原始 broker order ref 或资金明细。
- 未写 broker lake、provider lake、catalog、NAS 或远端 Git。
- `/tmp/runner-qmt-multifactor-operator-spec-20260625.json` 是一次性本地运行输入，不作为 process evidence。

## 6. 阶段决策

结论：`BLOCKED_EXTERNAL_RUNTIME_SESSION`。

工程侧 operator、持久化策略、稳定性窗口、测试和真实 gateway fail-closed 探测已完成。真实 P1-P4 simulation end-to-end 仍需 gateway session 刷新后复跑；当前不能宣称真实多因子模拟盘 submit/cancel 已完成。
