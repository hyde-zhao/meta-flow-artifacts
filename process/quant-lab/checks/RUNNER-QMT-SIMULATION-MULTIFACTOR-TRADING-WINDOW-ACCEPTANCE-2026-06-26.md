---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T09:34:43+08:00"
owner: "host-orchestrator"
status: "BLOCKED"
runtime_authorization_granted: true
runtime_authorization_ref: "runner-qmt-simulation-multifactor-trading-window-20260626-r1-auth"
run_id: "runner-qmt-simulation-multifactor-trading-window-2026-06-26-r1"
runtime_touched: true
small_live_or_live_authorized: false
---

# Runner QMT Simulation Multifactor Trading Window Acceptance

## 1. 结论

本次交易窗口 simulation runtime 验收未通过，结论为 `BLOCKED`。

| 项目 | 结果 |
|---|---|
| P0 gateway health | `PASS`，gateway 可达，`session_ready=true`，`runtime_status=xtquant-ready` |
| P0 capabilities | `PASS_WITH_CONTRACT_VIEW`，endpoint matrix 包含 positions、simulation submit、simulation cancel |
| P0.5 signed positions readonly | `PASS`，返回脱敏持仓摘要，`raw_payload_emitted=false`，`redaction_status=pass` |
| P1 target portfolio | `PASS`，target generated，`selected_count=1` |
| P2 order plan | `PASS`，order plan generated，`order_count=1` |
| P3 simulation submit/cancel | `BLOCKED`，submit action `transport_timeout`，`submitted_count=0`，`cancelled_count=0`，`unknown_count=0` |
| P4 reconciliation | `BLOCKED`，`unresolved_count=1`，`manual_takeover_required=true` |
| 稳定性窗口 | 不计数，本次不是 `1/5` |
| small_live / live | `NOT_AUTHORIZED`，未触达 |

## 2. 证据

| Evidence | 路径 |
|---|---|
| trading-window readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26.json` |
| runtime operator evidence | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r1/operator-evidence.json` |
| runtime operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r1/operator-evidence.index.json` |

所有 evidence 均只允许保留脱敏摘要、digest、bucket、instrument/order intent ref 和计数；不得保存 raw account、raw symbol、raw broker ref、raw payload、secret/token、fund detail。

## 3. 根因判断

本次阻断不是 gateway session、HMAC、profile 或 positions readonly 的整体失败。只读链路和 gateway health 均已通过。

直接触发点是 P3 submit 请求返回 `transport_timeout`。证据显示：

- execution action：`operation=submit`、`status=blocked`、`blocked_reason=transport_timeout`。
- `submitted_count=0`、`cancelled_count=0`、`unknown_count=0`。
- 用户人工核对 MiniQMT 未看到异常挂单。
- gateway 在失败后仍可 health pass，说明 gateway 进程未整体失效。

根因归类为输入契约问题：本次 runtime spec 仍继承非交易窗口 fixture 输入，正式 StrategyAdmissionPackage / operator spec 中 target symbol、current positions 和 risk positions 仍为 `INSTRUMENT_*` fixture 标识。P3 将 fixture instrument 送入真实 QMT simulation submit 路径后，submit HTTP 调用在 10 秒内未返回。

因此，本次不能视为“正式 StrategyAdmissionPackage + 可提交 simulation instrument + 授权窗口 current positions”的验收。

## 4. Manual Takeover / Fail Closed

| 项目 | 状态 |
|---|---|
| manual takeover required | `true` |
| 用户人工核对 | 用户反馈未看到异常挂单 |
| 自动重试 | 禁止，未执行第二次 submit/cancel |
| gateway stop | 未在本记录中确认；需 Windows 端手工停止后补充 |
| 下一步 runtime | 阻断，直到输入 contract 修正并重新获得逐次授权 |

## 5. 后续动作

1. 不计入稳定性窗口。
2. 不继续第二次 runtime submit/cancel。
3. 修正交易窗口 runtime 输入契约：不得用 `INSTRUMENT_*` fixture target 触达真实 QMT simulation submit。
4. 生成新的可提交 simulation instrument / current positions 脱敏输入方案后，重新申请逐次 runtime authorization。
5. Windows 端停止 gateway，并确认 `/qmt/health` 不可达或记录停止方式。
