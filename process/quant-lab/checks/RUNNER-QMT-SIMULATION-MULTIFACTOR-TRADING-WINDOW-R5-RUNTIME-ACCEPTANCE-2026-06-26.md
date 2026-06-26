---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R5-RUNTIME-ACCEPTANCE-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T11:25:00+08:00"
owner: "host-orchestrator"
status: "PASS_WITH_STABILITY_WINDOW_COLLECTING"
runtime_authorization_granted: true
runtime_authorization_ref: "runner-qmt-simulation-multifactor-trading-window-20260626-r5-auth"
run_id: "runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5"
runtime_touched: true
small_live_or_live_authorized: false
stability_window_completed_runs: 4
stability_window_required_runs: 5
---

# Runner QMT Simulation Multifactor Trading Window R5 Runtime Acceptance

## 1. 结论

用户已授权本次逐次 `simulation runtime authorization`。r5 按 Phase A 计划从 P0 gateway health / runtime identity 开始执行，最终结论为 `PASS_WITH_STABILITY_WINDOW_COLLECTING`。

本轮 r5 已通过：

- P0 gateway health / runtime identity：`status=ok`、`session_ready=true`、`runtime_status=xtquant-ready`、`runtime_mode=simulation`、`account_kind=simulation`、`runtime_profile=cr138-simulation`。
- P0.5 signed positions readonly：`status=ok`，positions 条数为 0，脱敏扫描未命中敏感字段。
- P1 target：通过。
- P2 order plan：通过，`order_count=1`。
- P3 simulation execution：通过，`submitted_count=1`、`cancelled_count=1`、`rejected_count=0`、`unknown_count=0`、`manual_takeover_required=false`。
- P4 reconciliation：通过，`unresolved_count=0`、`drift_bucket=drift:pass`、`manual_takeover_required=false`。

Phase A stability window 从 `3/5` 推进到 `4/5`。后续还需要 r6 一次独立逐次授权 runtime 成功，才能达到 `5/5` 并进入 Phase B simulation operational readiness closure。

本轮未授权、未执行 `small_live` / `live`，未触达 NAS、provider/lake/catalog、publish 或 remote Git。

## 2. 验收矩阵

| 阶段 | 结果 | 说明 |
|---|---|---|
| P0 gateway health | `PASS` | Windows gateway 在线，identity 为 simulation / CR138 |
| P0 capabilities | `PASS_WITH_CONTRACT_VIEW` | endpoint matrix 包含 positions、simulation submit、simulation cancel；unsigned capabilities authorization 字段不作为阻断 |
| R5 input generation | `PASS` | 私有 r5 runtime spec/admission 已生成；下单相关 symbol 无 `INSTRUMENT_*` 残留 |
| R5 plan-only | `PASS` | `runtime_touched=false`，`order_count=1` |
| P0.5 signed positions readonly | `PASS` | readonly evidence 写入成功，positions 条数为 0 |
| P1 target | `PASS` | target generated |
| P2 order plan | `PASS` | `order_count=1` |
| P3 execution | `PASS` | simulation submit/cancel 各 1 次；无 rejected / unknown；无需人工接管 |
| P4 reconciliation | `PASS` | `unresolved_count=0`，对账漂移桶为 pass |
| Stability window | `COLLECTING` | `completed_runs=4`、`required_runs=5` |
| Redaction | `PASS` | 正式 evidence 扫描未命中原始 ETF 代码、fixture instrument、账号字段、secret/password 或 raw payload |

## 3. 证据

| Evidence | 路径 |
|---|---|
| r5 P0 health evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-HEALTH-EVIDENCE-2026-06-26-r5.json` |
| r5 P0 capabilities evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-P0-CAPABILITIES-EVIDENCE-2026-06-26-r5.json` |
| r5 readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r5.json` |
| r5 runtime operator evidence | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.json` |
| r5 runtime evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.index.json` |
| r5 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r5.json` |
| r5 plan-only evidence | `process/evidence/runner-simulation-runtime-input-fix-2026-06-26/plan-only-r5/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.json` |
| r5 private runtime overlay | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5-runtime-overlay.json` |
| r5 private runtime spec | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5-runtime-spec.json` |
| r5 private runtime admission | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5-runtime-admission-package.json` |

## 4. 计数修正说明

r5 operator evidence 的 runtime 行为是 PASS，但其内嵌 `stability_window.completed_runs` 字段显示 5。原因是 r5 runtime spec 把本轮 readonly evidence ref 写进了 `stability_evidence_refs`，operator 又在当前 runtime PASS 后额外加 1，造成过计数。

处理方式：

1. 不重跑 r5 runtime，避免在同一授权下重复 submit/cancel。
2. 以 Phase A 已批准 baseline `3/5` 加本轮 r5 PASS，生成独立 stability window summary，结论为 `4/5`。
3. 已再次修复 `trading/strategy_runner/runtime_inputs.py`：`stability_evidence_refs` 只保存历史成功 run refs，不再追加当前 readonly evidence ref。
4. 已同步修复到 `/mnt/c/quant-lab-runtime/trading/strategy_runner/runtime_inputs.py`；该 builder 修复不要求重启 gateway。
5. 定向回归 `tests/test_runner_runtime_inputs.py tests/test_runner_multifactor_simulation_operator.py tests/test_cr020_runtime_manual_validation.py` 通过，`18 passed`。

## 5. 脱敏与边界

- 正式 evidence 不保存 raw account、raw symbol、raw broker order ref、raw payload、secret/token 或 fund detail。
- r5 evidence 扫描未命中私有 ETF 原始代码、`INSTRUMENT_*`、账号字段、secret/password 或 `raw_payload=true`。
- 私有 runtime spec/admission 位于 `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/`，允许包含真实可提交 symbol；这些私有输入不是正式 evidence。
- 本轮授权只覆盖 r5 simulation runtime，不覆盖 r6，也不覆盖 `small_live` / `live`。

## 6. 后续动作

| 动作 | 状态 | 说明 |
|---|---|---|
| Phase A r5 | `PASS` | stability window 推进到 `4/5` |
| Phase A r6 | `PENDING_AUTHORIZATION` | 需要下一次逐次 simulation runtime authorization，从 P0 开始 |
| Phase B simulation readiness closure | `PENDING_5_OF_5` | 等 `5/5` 后再收口 |
| Phase C small_live/live preparation | `DEFERRED` | 必须独立 CR、独立人工决策和独立 runtime authorization |
