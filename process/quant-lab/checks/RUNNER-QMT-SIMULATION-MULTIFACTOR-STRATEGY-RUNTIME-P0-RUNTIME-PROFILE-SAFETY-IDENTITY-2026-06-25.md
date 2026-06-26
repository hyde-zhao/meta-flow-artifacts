---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P0-RUNTIME-PROFILE-SAFETY-IDENTITY-2026-06-25"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T11:32:04+08:00"
owner: "host-orchestrator"
phase_goal: "P0-runtime-profile-and-safety-identity"
stage_decision: "PASS"
target_status: "runtime-profile-safety-identity-ready"
evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P0-EVIDENCE-2026-06-25.json"
runtime_authorization_granted: false
---

# P0 Runtime Profile 与安全身份识别检查结果

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `PASS` |
| 当前 P0 状态 | `runtime-profile-safety-identity-ready` |
| 下一阶段 | `P1-multifactor-target-portfolio` |
| Evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P0-EVIDENCE-2026-06-25.json` |

P0 已完成离线合同和测试：simulation client 请求必须携带 `expected_runtime_mode=simulation` 与 `expected_runtime_profile`；gateway runtime 必须声明 `QMT_RUNTIME_MODE`、`QMT_ACCOUNT_KIND` 和 `QMT_RUNTIME_PROFILE`；缺失或不匹配时在 XtQuant adapter 调用前 fail closed。

本检查不授权真实 gateway runtime、账户查询、submit/cancel、cancel-all、凭据读取、NAS、provider/lake/catalog、远端写入或 `small_live` / `live`。

## 2. Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 上游 simulation smoke 已通过 | PASS | `process/checks/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-VERIFICATION-2026-06-24.md` | buy/sell simulation submit/cancel 已通过 |
| P0 目标明确 | PASS | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-PHASE-GOALS-2026-06-25.md` | P0 要先固化 runtime profile 与身份识别 |
| 不授权边界明确 | PASS | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-START-2026-06-25.md` | 不触发真实 runtime / live |

## 3. Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | Client 请求侧 expected runtime 字段存在 | PASS | `trading/qmt_client.py` | `QmtClientConfig`、`QmtRequest`、`QmtRestRequest` 均携带 expected mode/profile |
| 2 | REST 请求体包含 expected runtime 字段 | PASS | `tests/test_cr020_linux_client_rest_transport.py` | 测试断言 body 包含 `expected_runtime_mode` 与 `expected_runtime_profile` |
| 3 | Gateway runtime 身份字段存在 | PASS | `trading/qmt_runtime.py` | `QMT_RUNTIME_MODE`、`QMT_ACCOUNT_KIND`、`QMT_RUNTIME_PROFILE` 进入 config public dict |
| 4 | 缺少 expected identity 时 client fail closed | PASS | `tests/test_cr020_linux_client_rest_transport.py` | transport 未被调用，禁止计数保持 0 |
| 5 | Gateway identity 缺失时 fail closed | PASS | `tests/test_cr020_runtime_manual_validation.py` | 阻断原因 `runtime_profile_mismatch`，adapter 未提交订单 |
| 6 | expected profile 与 gateway profile 不匹配时 fail closed | PASS | `tests/test_cr020_runtime_manual_validation.py` | 阻断原因 `runtime_profile_mismatch`，adapter 未提交订单 |
| 7 | 既有 simulation submit/cancel 成功路径保持 | PASS | `tests/test_cr020_runtime_manual_validation.py` | 明确 simulation identity 后 fake XtQuant submit/cancel 通过 |
| 8 | 不触发真实 runtime | PASS | 本轮命令记录 | 只运行 pytest / py_compile，未访问 gateway、账户、订单或凭据 |

## 4. Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| runtime profile handshake 合同可执行 | PASS | `trading/qmt_client.py` / `trading/qmt_runtime.py` | 请求侧和 gateway 侧都有身份字段 |
| mismatch fail closed | PASS | 新增测试 | mismatch 在 adapter 调用前阻断 |
| 验证通过 | PASS | `25 passed` + py_compile | 受影响测试全部通过 |
| 下一阶段可启动 | PASS | 本检查结论 | 可进入 P1 多因子目标组合；P1 不下单、不触达 QMT runtime |

## 5. Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Gateway 合同 | `trading/qmt_gateway_contracts.py` | PASS | 新增 runtime identity expectation 与阻断原因 |
| Client 合同 | `trading/qmt_client.py` | PASS | simulation transport 显式 expected identity |
| Runtime gateway | `trading/qmt_runtime.py` | PASS | runtime identity config 与 fail-closed 检查 |
| Transport metadata | `trading/qmt_transport.py` | PASS | expected identity 纳入 REST metadata 白名单 |
| 测试 | `tests/test_cr020_runtime_manual_validation.py` / `tests/test_cr020_linux_client_rest_transport.py` | PASS | 覆盖成功、缺失、mismatch |
| Evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P0-EVIDENCE-2026-06-25.json` | PASS | 结构化证据 |

## 6. 验证命令

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr020_runtime_manual_validation.py tests/test_cr020_linux_client_rest_transport.py tests/test_runner_simulation_activation.py` | PASS | `25 passed in 0.27s` |
| `PYTHONPYCACHEPREFIX=/tmp/p0-runtime-profile-pycompile uv run --python 3.11 python -m py_compile trading/qmt_gateway_contracts.py trading/qmt_client.py trading/qmt_runtime.py trading/qmt_transport.py tests/test_cr020_runtime_manual_validation.py tests/test_cr020_linux_client_rest_transport.py tests/test_runner_simulation_activation.py` | PASS | 无语法错误 |

## 7. 阶段决策

结论：`PASS`

P0 已处理完成。下一步建议进入 P1：多因子策略生成 target portfolio。P1 仍只生成可审计目标组合，不下单、不触达 QMT gateway、不读取凭据，不继承 simulation smoke 的 runtime 授权。
