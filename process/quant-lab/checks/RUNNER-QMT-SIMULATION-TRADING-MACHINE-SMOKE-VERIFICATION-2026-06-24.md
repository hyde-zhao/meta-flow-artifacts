---
check_id: "RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-VERIFICATION-2026-06-24"
task_id: "RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-2026-06-24"
created_at: "2026-06-25T09:01:24+08:00"
owner: "meta-qa-critical"
source_status: "simulation-entry-ready"
target_status: "simulation-smoke-passed"
stage_decision: "PASS"
state_transition_applied: true
evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json"
---

# Runner QMT Simulation Trading Machine Smoke Verification

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `PASS` |
| 当前状态 | `simulation-smoke-passed` |
| 目标状态 | `simulation-smoke-passed` |
| 状态推进 | 已推进：`simulation-entry-ready` -> `simulation-smoke-passed` |
| 路由 | `completed` |
| Evidence | `process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json` |

本轮完成了允许边界内的 smoke 准备验证：本地 fake XtQuant / runtime 合同、HMAC scope、simulation submit/cancel 脱敏断言、编译和 diff 检查均通过。

2026-06-25 09:32:35 +08:00 收到用户 runtime 授权后，Host Orchestrator 对已启动的交易机 gateway 做了实机探测。前两轮因 runtime 配置未加载阻塞；2026-06-25 09:53:54 +08:00 gateway 达到 `session_ready=true` / `xtquant-ready`。2026-06-25 10:05:34 +08:00 signed positions readonly 通过；signed simulation submit endpoint 的 auth/session 到达验证通过，并被故意无效 payload 在 validation 阶段拦截，确认未触达下单 adapter。

2026-06-25 10:38:00 +08:00，在用户确认模拟委托已手工撤销、gateway 已从 `C:/quant-lab-runtime` 重启后，Host Orchestrator 复跑 buy 侧 simulation submit/cancel smoke。signed buy submit 返回 `status=ok`、`accepted=true`、`adapter_status=xtquant-simulation-submit-accepted`，并返回非原文 `cancel-ref`；随后 signed cancel 使用该 `cancel-ref` 返回 `status=ok`、`accepted=true`、`adapter_status=xtquant-simulation-cancel-accepted`。

2026-06-25 10:43:00 +08:00，按用户追加授权补测 sell 侧 simulation submit/cancel smoke。signed sell submit 返回 `status=ok`、`accepted=true`、`adapter_status=xtquant-simulation-submit-accepted`，并返回非原文 `cancel-ref`；随后 signed cancel 使用该 `cancel-ref` 返回 `status=ok`、`accepted=true`、`adapter_status=xtquant-simulation-cancel-accepted`。响应和 evidence 未保存凭据、账号原文、证券代码原文、原始 broker order ref 或资金明细。

## 2. 读取文件

| 类型 | 文件 |
|---|---|
| handoff | `process/context/RUNNER-SIMULATION-TRADING-MACHINE-SMOKE-HANDOFF-2026-06-24.md` |
| work package | `process/context/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-WORK-PACKAGE-2026-06-24.md` |
| upstream verification | `process/checks/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-VERIFICATION-2026-06-24.md` |
| source | `trading/strategy_runner/simulation_activation.py` |
| source | `trading/qmt_client.py` |
| source | `trading/qmt_gateway_contracts.py` |
| source | `trading/qmt_runtime.py` |
| tests | `tests/test_runner_simulation_activation.py` |
| tests | `tests/test_cr020_runtime_manual_validation.py` |

## 3. 验证范围

本轮验证范围：

- 模型盘目标组合到 simulation order 激活入口的 stage gate / risk / gateway 边界。
- QMT client `allow_simulation_transport` 默认 fail-closed 与显式开启后的 REST 合同。
- QMT runtime simulation submit/cancel 的 HMAC scope、session gate、fake XtQuant adapter、脱敏 response。
- 交易机现场 simulation buy submit/cancel 与 sell submit/cancel 双向 side 覆盖。
- 脱敏 evidence 输出边界。

本轮不授权也未执行：

- 真实资金 `small_live` / `live` submit。
- 真实资金 cancel。
- 真实账户资金或持仓原文保存。
- 原始证券代码、broker order ref、账号、secret、token、资金明细写入证据。
- 放宽 simulation scope 到 live scope。

## 4. 验证命令与结果

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_runner_simulation_activation.py tests/test_cr020_runtime_manual_validation.py tests/test_cr020_linux_client_rest_transport.py` | PASS | `21 passed`；覆盖 fake XtQuant、activation、runtime HMAC scope、REST transport、脱敏断言 |
| `PYTHONPYCACHEPREFIX=/tmp/runner-qmt-simulation-smoke-pycompile uv run --python 3.11 python -m py_compile trading/qmt_client.py trading/strategy_runner/simulation_activation.py trading/qmt_gateway_contracts.py trading/qmt_runtime.py tests/test_runner_simulation_activation.py tests/test_cr020_runtime_manual_validation.py tests/test_cr020_linux_client_rest_transport.py` | PASS | 无语法错误输出 |
| `git diff --check` | PASS | 无空白错误输出 |
| `curl -sS --max-time 5 http://172.30.32.1:18765/qmt/health` | BLOCKED | gateway 可达；`session_ready=false`，`blocked_reason=credential_not_configured`，`runtime_status=missing-miniqmt-path-or-account` |
| `curl -sS --max-time 5 http://172.30.32.1:18765/qmt/capabilities` | PASS | capabilities 可达；simulation submit/cancel 仍为 later-gated contract endpoint |
| `curl -sS --max-time 5 http://127.0.0.1:18765/qmt/health` | INFO | WSL 本机 loopback 无服务；Windows host 网关地址为 `172.30.32.1:18765` |
| `curl -sS --max-time 5 http://172.30.32.1:18765/qmt/health`（2026-06-25 09:45:26 +08:00 复测） | BLOCKED | gateway 重启后仍可达；仍为 `session_ready=false`、`blocked_reason=credential_not_configured`、`runtime_status=missing-miniqmt-path-or-account` |
| `curl -sS --max-time 5 http://172.30.32.1:18765/qmt/capabilities`（2026-06-25 09:45:26 +08:00 复测） | PASS | capabilities 仍可达；simulation submit/cancel 未执行 |
| `curl -sS --max-time 5 http://172.30.32.1:18765/qmt/health`（2026-06-25 09:53:54 +08:00 复测） | PASS | gateway 可达；`session_ready=true`，`runtime_status=xtquant-ready`，MiniQMT/account/HMAC/WSL allowlist 已配置 |
| `curl -sS --max-time 5 http://172.30.32.1:18765/qmt/capabilities`（2026-06-25 09:53:54 +08:00 复测） | PASS | capabilities 仍可达；simulation submit/cancel 未执行 |
| `uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli query-positions ...` | PASS | signed positions readonly 返回 `status=ok`；`position_count=1`，`raw_payload_emitted=false`，`redaction_status=pass` |
| signed simulation submit validation request | PASS_WITH_EXPECTED_BLOCK | signed request 到达 simulation submit endpoint；因故意缺少 `symbol` 被 `validation_blocked` 拦截；`real_order=0`、`qmt_operation=0`、`simulation_or_live_run=0` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr020_runtime_manual_validation.py tests/test_cr020_linux_client_rest_transport.py tests/test_runner_simulation_activation.py` | PASS | `22 passed`；覆盖 cancel-ref、数字型 XtQuant order_id 撤单兼容、fake XtQuant、activation、runtime HMAC scope、REST transport 和脱敏断言 |
| `PYTHONPYCACHEPREFIX=/tmp/qmt-runtime-fix-pycompile uv run --python 3.11 python -m py_compile trading/qmt_runtime.py trading/qmt_gateway_contracts.py trading/qmt_client.py tests/test_cr020_runtime_manual_validation.py` | PASS | 回修后无语法错误输出 |
| final signed buy simulation submit/cancel smoke | PASS | buy submit accepted，cancel accepted；cancel adapter 返回 `xtquant-simulation-cancel-accepted`；只保存脱敏摘要 |
| final signed sell simulation submit/cancel smoke | PASS | sell submit accepted，cancel accepted；cancel adapter 返回 `xtquant-simulation-cancel-accepted`；只保存脱敏摘要 |

早期实机探测只访问无 HMAC 的 health / capabilities endpoint；最终 smoke 使用 signed HMAC client 执行 positions readonly、simulation submit 和 simulation cancel。全程未读取或回显交易机 env 原文，未保存账号原文、证券代码原文、原始订单引用、secret、token 或资金明细。

## 5. 设计契约验证

| 契约 | 来源 | 验证方式 | 结果 |
|---|---|---|---|
| simulation submit/cancel 必须使用独立 endpoint 与 scope | `trading/qmt_gateway_contracts.py` / `trading/qmt_runtime.py` | 源码审查 + pytest HMAC scope 用例 | PASS |
| client 默认不得走 simulation transport | `trading/qmt_client.py` | 源码审查；`allow_simulation_transport=False` 默认值存在 | PASS |
| stage gate 缺 runbook / kill switch / reconciliation 时不得触达 gateway | `trading/strategy_runner/simulation_activation.py` / tests | pytest activation 阻断用例 | PASS |
| pretrade risk 失败时不得触达 gateway | `trading/strategy_runner/simulation_activation.py` / tests | pytest activation 阻断用例 | PASS |
| runtime response 不输出账号、原始证券代码、原始 broker order ref 或 secret | `trading/qmt_gateway_contracts.py` / tests | pytest JSON render 脱敏断言 | PASS |
| 真实 simulation submit/cancel 需要 QMT session ready 与 authorized HMAC scope | `trading/qmt_runtime.py` | 源码审查 + pytest auth scope 用例 | PASS |

## 6. 阻塞项

| ID | 阻塞项 | 原因 | 当前处理 |
|---|---|---|---|
| B-001 | 缺少 per-run runtime authorization | 启动/连接 QMT gateway、绑定端口、读取 runtime 配置、账户只读查询、simulation submit/cancel 均属于真实 runtime 操作 | 已解决：用户授权本轮模拟账户 smoke；不包含 small_live/live |
| B-002 | 当前会话不得读取或保存凭据/账号/订单原文 | 用户明确禁止保存或推送凭据、账号原文、订单原文、secret、token、资金明细原文 | 未读取、未保存 |
| B-003 | 未确认交易机 MiniQMT/XtQuant/gateway 已就绪 | 当前执行环境无法证明真实 QMT session ready | 未启动、未连接 |
| B-004 | gateway 已启动但 runtime 配置未加载 | health 显示 MiniQMT 路径、账号引用、client_id、client_secret 均未配置，session 未 ready | 2026-06-25 09:53:54 已解决，health 显示 `session_ready=true` |
| B-005 | WSL 侧 HMAC client env 不可见 | `/mnt/c/Users/hydez/.quant-lab/runtime/qmt/cr138-simulation/client.env` 在 WSL 不存在或不可见；没有 client secret 不能生成签名 POST | 2026-06-25 10:05:34 已解决，`/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/client.env` 可用且权限为 `600` |
| B-006 | submit/cancel 取消引用契约缺口 | submit response 只返回脱敏 broker ref；cancel endpoint 需要原始 broker ref。直接执行 submit 后无法保证可撤单 | 已解决：gateway 使用非原文 `cancel-ref` 映射原始 broker ref，并兼容数字型 XtQuant order id 撤单 |

## 7. 最小授权清单

用户已授权本轮仅限模拟账户 smoke。交易机 gateway 已在 2026-06-25 09:53:54 +08:00 达到 `session_ready=true`，WSL 侧 HMAC client env 已在 2026-06-25 10:05:34 +08:00 验证可用。2026-06-25 10:38:00 +08:00 buy 侧 simulation submit/cancel smoke 已通过；2026-06-25 10:43:00 +08:00 sell 侧 simulation submit/cancel smoke 已通过。不要把真实 broker order ref 输出给 client 或 evidence。

- submit response 应返回可用于 cancel 的非原文 cancel token，或 gateway 在内存中维护脱敏 broker ref 到原始 broker ref 的短期映射。
- cancel request 不得要求 client 保存或回传原始 broker order ref。
- 双向 side smoke 已完成；不得基于本授权进入 `small_live` / `live`。

## 8. Runtime 探测结果

| 项目 | 结果 |
|---|---|
| gateway base URL | `http://172.30.32.1:18765` |
| health | `PASS` |
| session_ready | `true` |
| blocked_reason | 空 |
| runtime_status | `xtquant-ready` |
| 09:45 复测 | 仍为 `BLOCKED`，重启后配置未加载 |
| 09:53 复测 | `PASS`，`session_ready=true`，`runtime_status=xtquant-ready` |
| 10:05 复测 | signed positions readonly `PASS`；signed simulation submit endpoint validation `PASS_WITH_EXPECTED_BLOCK` |
| 10:31 复测 | simulation submit `PASS`，返回 `cancel-ref`；simulation cancel `BLOCKED`，`validation_blocked`，未触达撤单 adapter |
| 10:38 复测 | buy simulation submit `PASS`；buy simulation cancel `PASS`，`xtquant-simulation-cancel-accepted` |
| 10:43 复测 | sell simulation submit `PASS`；sell simulation cancel `PASS`，`xtquant-simulation-cancel-accepted` |
| 当前阻塞 | 无 |
| positions readonly | 已执行，通过，仅保存脱敏摘要 |
| simulation buy submit/cancel | 已执行，通过，仅保存脱敏摘要 |
| simulation sell submit/cancel | 已执行，通过，仅保存脱敏摘要 |
| 原文凭据 / 账号 / 订单保存 | 未发生 |

## 9. 下一步要求

```text
本阶段 simulation smoke 已通过。不得基于本结果进入 small_live/live；如需进入小额实盘，必须另起高风险授权门、独立 endpoint/scope、限额、kill switch、cancel-only 和人工确认。
```

## 10. 阶段决策

结论：`PASS`

允许边界内的本地准备验证、gateway health、positions readonly、simulation buy submit/cancel 和 simulation sell submit/cancel 均通过。状态保持 `simulation-smoke-passed`。本结论不授权真实资金 `small_live` / `live` submit/cancel，不授权保存或推送凭据、账号原文、证券代码原文、原始订单引用或资金明细。
