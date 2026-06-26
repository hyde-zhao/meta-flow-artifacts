---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R2-REVALIDATION-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T10:00:00+08:00"
owner: "host-orchestrator"
status: "BLOCKED_EXPECTED_FAIL_CLOSED"
runtime_authorization_granted: true
runtime_authorization_ref: "runner-qmt-simulation-multifactor-trading-window-20260626-r2-auth"
run_id: "runner-qmt-simulation-multifactor-trading-window-2026-06-26-r2"
runtime_touched: true
small_live_or_live_authorized: false
---

# Runner QMT Simulation Multifactor Trading Window R2 Revalidation

## 1. 结论

用户重启 Windows gateway 后，已重新执行 r2 runtime 验证。结论为 `BLOCKED_EXPECTED_FAIL_CLOSED`：

- gateway / session / runtime identity 已恢复正常。
- signed positions readonly 通过。
- P1 target 与 P2 order plan 仍可生成。
- P3 submit 不再表现为 `transport_timeout`，而是在 submit 阶段返回 `validation_blocked`。
- `submitted_count=0`、`cancelled_count=0`、`unknown_count=0`。
- 业务验收仍不通过，稳定性窗口不计数。

本轮证明 gateway 侧防护补丁已生效：fixture instrument 不再进入 XtQuant `order_stock` 后等待超时，而是 fail closed。

## 2. R2 验证矩阵

| 阶段 | 结果 | 说明 |
|---|---|---|
| P0 gateway health | `PASS` | `runtime_mode=simulation`、`account_kind=simulation`、`runtime_profile=cr138-simulation`、`session_ready=true` |
| P0 capabilities | `PASS_WITH_CONTRACT_VIEW` | endpoint matrix 包含 positions、simulation submit、simulation cancel |
| P0.5 signed positions readonly | `PASS` | 1 条脱敏 item，`raw_payload_emitted=false`，`redaction_status=pass` |
| P1 target | `PASS` | target generated，`selected_count=1` |
| P2 order plan | `PASS` | order plan generated，`order_count=1` |
| P3 execution | `BLOCKED` | action `blocked_reason=validation_blocked` |
| P4 reconciliation | `BLOCKED` | `unresolved_count=1`，`manual_takeover_required=true` |

## 3. 证据

| Evidence | 路径 |
|---|---|
| r2 readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r2.json` |
| r2 runtime operator evidence | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r2/operator-evidence.json` |
| r2 runtime evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r2/operator-evidence.index.json` |

## 4. 根因状态

根因仍为正式 runtime 输入契约未满足交易窗口验收要求：

- `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json` 的 `current_positions` 仍为 `INSTRUMENT_FIXTURE_A/B`。
- formal StrategyAdmissionPackage 的 target symbol 仍为 fixture 标识。
- r2 私有 spec 继承该输入，因此不能通过真实 QMT simulation submit。

本轮不能计入稳定性窗口；下一步不是继续重试 runtime，而是修正 formal StrategyAdmissionPackage / operator spec 的 runtime 输入生成契约。

## 5. 后续动作

1. 不再重试当前 fixture spec 的 runtime submit/cancel。
2. Windows gateway 可停止；若继续调试，仍需逐次授权。
3. 修正交易窗口 runtime 输入契约：P3 只能接收真实可提交 simulation instrument，不允许 `INSTRUMENT_*`、`instrument:`、`symbol:` 或 fixture ref 进入 gateway submit。
4. 修正后重新生成正式 StrategyAdmissionPackage/operator spec，并重新走逐次 runtime authorization。
