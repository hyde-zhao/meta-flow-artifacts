---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R6-RUNTIME-ACCEPTANCE-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T11:30:00+08:00"
owner: "host-orchestrator"
status: "PASS_STABILITY_WINDOW_COMPLETE"
runtime_authorization_granted: true
runtime_authorization_ref: "runner-qmt-simulation-multifactor-trading-window-20260626-r6-auth"
run_id: "runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6"
runtime_touched: true
small_live_or_live_authorized: false
stability_window_completed_runs: 5
stability_window_required_runs: 5
---

# Runner QMT Simulation Multifactor Trading Window R6 Runtime Acceptance

## 1. 结论

用户已授权本次逐次 `simulation runtime authorization`。r6 按 Phase A 计划从 P0 gateway health / runtime identity 开始执行，最终结论为 `PASS_STABILITY_WINDOW_COMPLETE`。

本轮 r6 已通过：

- P0 gateway health / runtime identity：`status=ok`、`session_ready=true`、`runtime_status=xtquant-ready`、`runtime_mode=simulation`、`account_kind=simulation`、`runtime_profile=cr138-simulation`。
- P0.5 signed positions readonly：`status=ok`，positions 条数为 0，脱敏扫描未命中敏感字段。
- P1 target：通过。
- P2 order plan：通过，`order_count=1`。
- P3 simulation execution：通过，`submitted_count=1`、`cancelled_count=1`、`rejected_count=0`、`unknown_count=0`、`manual_takeover_required=false`。
- P4 reconciliation：通过，`unresolved_count=0`、`drift_bucket=drift:pass`、`manual_takeover_required=false`。

Phase A stability window 已从 `4/5` 推进到 `5/5`，达到要求。下一步不应继续重复 runtime submit/cancel；应进入 Phase B simulation operational readiness closure，汇总 `5/5` evidence 并形成 simulation readiness 结论。

本轮未授权、未执行 `small_live` / `live`，未触达 NAS、provider/lake/catalog、publish 或 remote Git。

## 2. 验收矩阵

| 阶段 | 结果 | 说明 |
|---|---|---|
| P0 gateway health | `PASS` | Windows gateway 在线，identity 为 simulation / CR138 |
| P0 capabilities | `PASS_WITH_CONTRACT_VIEW` | endpoint matrix 包含 positions、simulation submit、simulation cancel；unsigned capabilities authorization 字段不作为阻断 |
| R6 input generation | `PASS` | 私有 r6 runtime spec/admission 已生成；历史 stability refs 为 4 条 |
| R6 plan-only | `PASS` | `runtime_touched=false`，`order_count=1` |
| P0.5 signed positions readonly | `PASS` | readonly evidence 写入成功，positions 条数为 0 |
| P1 target | `PASS` | target generated |
| P2 order plan | `PASS` | `order_count=1` |
| P3 execution | `PASS` | simulation submit/cancel 各 1 次；无 rejected / unknown；无需人工接管 |
| P4 reconciliation | `PASS` | `unresolved_count=0`，对账漂移桶为 pass |
| Stability window | `PASS` | `completed_runs=5`、`required_runs=5`，operator 内嵌计数与 rollup 一致 |
| Redaction | `PASS` | 正式 evidence 扫描未命中原始 ETF 代码、fixture instrument、账号字段、secret/password 或 raw payload |

## 3. 证据

| Evidence | 路径 |
|---|---|
| r6 P0 health evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-HEALTH-EVIDENCE-2026-06-26-r6.json` |
| r6 P0 capabilities evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-CAPABILITIES-EVIDENCE-2026-06-26-r6.json` |
| r6 readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r6.json` |
| r6 runtime operator evidence | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.json` |
| r6 runtime evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.index.json` |
| r6 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r6.json` |
| r6 plan-only evidence | `process/evidence/runner-simulation-runtime-input-fix-2026-06-26/plan-only-r6/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.json` |
| r6 private runtime overlay | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6-runtime-overlay.json` |
| r6 private runtime spec | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6-runtime-spec.json` |
| r6 private runtime admission | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6-runtime-admission-package.json` |

## 4. Stability Window Summary

| 字段 | 值 |
|---|---|
| required_runs | 5 |
| completed_runs | 5 |
| status | `pass` |
| current run | r6 |
| submitted_count | 1 |
| cancelled_count | 1 |
| rejected_count | 0 |
| unknown_count | 0 |
| unresolved_count | 0 |
| manual_takeover_required | false |

5 条成功运行证据引用：

1. `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-2026-06-25.json`
2. `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.index.json`
3. `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r4/operator-evidence.index.json`
4. `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.index.json`
5. `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.index.json`

## 5. 脱敏与边界

- 正式 evidence 不保存 raw account、raw symbol、raw broker order ref、raw payload、secret/token 或 fund detail。
- r6 evidence 扫描未命中私有 ETF 原始代码、`INSTRUMENT_*`、账号字段、secret/password 或 `raw_payload=true`。
- 私有 runtime spec/admission 位于 `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/`，允许包含真实可提交 symbol；这些私有输入不是正式 evidence。
- 本轮授权只覆盖 r6 simulation runtime，不覆盖后续重复 runtime，也不覆盖 `small_live` / `live`。

## 6. 后续动作

| 动作 | 状态 | 说明 |
|---|---|---|
| Phase A r6 | `PASS` | stability window 达到 `5/5` |
| Phase B simulation readiness closure | `READY_TO_START` | 汇总 5/5 evidence，形成 simulation readiness 结论和后续门禁 |
| 继续重复 simulation runtime | `NOT_RECOMMENDED` | 稳定窗口已满，不应在无新目标下重复 submit/cancel |
| Phase C small_live/live preparation | `DEFERRED` | 必须独立 CR、独立人工决策和独立 runtime authorization |
