---
context_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-PHASE-GOALS-AND-PLAN-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T11:10:00+08:00"
owner: "host-orchestrator"
status: "active-context-entry"
current_phase: "story-execution"
runtime_authorization_granted: false
simulation_runtime_authorization_required_per_run: true
small_live_or_live_authorized: false
---

# Runner QMT Simulation Multifactor Next Phase Goals And Plan

## 1. 目的

本文档作为后续恢复入口，统一记录 `RUNNER-QMT-SIMULATION-MULTIFACTOR` 在 r3 交易窗口验收后的阶段目标、计划、授权边界和下一步执行顺序。

它服务三个后续方向：

1. 继续模拟盘 stability window 累计。
2. 在稳定窗口完成后收口 simulation operational readiness。
3. 为未来 `small_live` / `live` 切换准备独立 CR 和决策输入。

本文档不是 runtime authorization，不授权继续下单、撤单、读取凭据、读取 raw account / raw order / fund detail，也不授权 `small_live` / `live`。

## 2. 当前状态快照

| 项目 | 状态 |
|---|---|
| Meta Flow phase | `story-execution` |
| Blocked | `false` |
| Active CR | none |
| Active Story | none |
| CR138 fixture/static baseline | CP8 已以 `READY_WITH_RISK` 关闭 |
| 真实 QMT simulation runtime | r3 交易窗口验收已通过 |
| Stability window | `completed_runs=5/5`，`pass` |
| `small_live` / `live` | deferred，未授权、未实现、未验证 |

### 2.1 r3 runtime 结论

| 阶段 | 结论 | 关键结果 |
|---|---|---|
| P0 gateway health / identity | PASS | `runtime_mode=simulation`、`account_kind=simulation`、`runtime_profile=cr138-simulation`、`session_ready=true` |
| P0.5 signed positions readonly | PASS | positions 条数为 0，evidence 脱敏 |
| P1 target | PASS | target 生成通过 |
| P2 order plan | PASS | `order_count=1` |
| P3 execution | PASS | simulation submit/cancel 各 1 次，`unknown_count=0` |
| P4 reconciliation | PASS | `unresolved_count=0`、`manual_takeover_required=false` |

### 2.2 关键证据

| 类型 | 路径 |
|---|---|
| r3 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R3-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r3 readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r3.json` |
| r3 operator evidence | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.json` |
| r3 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.index.json` |
| r4 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R4-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r4 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r4.json` |
| r4 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r4/operator-evidence.index.json` |
| r5 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R5-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r5 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r5.json` |
| r5 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.index.json` |
| r6 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R6-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r6 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r6.json` |
| r6 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.index.json` |
| Phase B readiness closure | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-OPERATIONAL-READINESS-CLOSURE-2026-06-26.md` |
| Phase B release context | `process/release/RELEASE-CONTEXT-RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-READINESS-2026-06-26.yaml` |
| Phase B human decision | `process/checkpoints/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-READINESS-HUMAN-DECISION-2026-06-26.md` |
| Ops policy | `process/policies/RUNNER-QMT-SIMULATION-MULTIFACTOR-RUNTIME-INPUT-GATEWAY-LIFECYCLE-POLICY-2026-06-26.md` |
| Ops policy closure | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-OPS-POLICY-CLOSURE-2026-06-26.md` |
| deferred live switch scenario pack | `process/context/RUNNER-QMT-SIMULATION-TO-LIVE-SWITCH-DEFERRED-SCENARIO-PACK-2026-06-25.md` |

## 3. 后续阶段目标

### Phase A: Simulation Stability Window Completion

目标：把 simulation runtime 稳定性窗口补到 `5/5`。当前已完成。

完成条件：

- 5 次独立 simulation runtime 成功记录已具备。
- 每次执行前都取得新的逐次 `simulation runtime authorization`。
- 每次都从 P0 gateway health / runtime identity 开始。
- 每次 signed positions readonly 通过。
- 每次 P1 / P2 / P3 / P4 通过。
- 每次 `unknown_count=0`。
- 每次 `unresolved_count=0`。
- 每次 `manual_takeover_required=false`。
- 每次 evidence 脱敏扫描通过。
- 每次都记录 evidence index 和验收摘要。

非目标：

- 不启动 `small_live` / `live`。
- 不复用上一轮 runtime authorization。
- 不保存 raw account、raw symbol、raw broker order ref、raw payload、secret/token 或 fund detail。
- 不触达 NAS、provider/lake/catalog、publish 或 remote Git。

### Phase B: Simulation Operational Readiness Closure

目标：在 stability window 达到 `5/5` 后，把 simulation runtime readiness 收口为可审计结论。

完成条件：

- 汇总 `5/5` simulation runtime evidence。已完成。
- 生成或更新 simulation runtime verification summary。已完成。
- 更新 `process/STATE.md` 和 `process/state/STATE.current.json`。已完成。
- 更新或补充 operational runbook。已完成。
- 明确 simulation readiness 的范围和非范围。已完成。
- 明确仍不包含 `small_live` / `live`。已完成。
- 必要时发起 CP7 / CP8 等价人工确认或 release readiness 更新。当前结论为 `READY_WITH_RISK`，且用户已接受；第三个交易日观察保留为可选后续，不再阻塞 simulation readiness closure。
- 固化 `SIM-OPS-GATE-02` runtime input policy。已完成。
- 固化 `SIM-OPS-GATE-03` gateway lifecycle policy。已完成。

非目标：

- 不把 simulation readiness 自动升级为 live readiness。
- 不创建或启用 live gateway route。
- 不读取 live credential。
- 不触达真实资金相关 raw detail。

### Phase C: Small Live / Live Switch Preparation

目标：为未来 `small_live` / `live` 切换准备独立 CR 和人工决策输入。

完成条件：

- 创建独立 live switch CR 或 change package。
- 将 `DQ-LIVE-001..004` 转成正式人工决策项。
- 明确 live gateway 是否使用独立 profile / port / env。
- 明确实盘前需要的 simulation stability window 和审批门槛。
- 明确首个实盘范围只能是 `small_live`，且包含 symbol、side、quantity、notional、frequency、max order count 等限制。
- 明确 live evidence 只允许保存脱敏 summary、count、digest、ref 和风险接受记录。
- 明确 cancel-only rollback、stop-new-orders、manual takeover 和 incident path。

非目标：

- 不在本文档内实现 live submit/cancel。
- 不在本文档内激活 `small_live` / `live` route。
- 不把 simulation authorization 复用为 live authorization。
- 不把 r3 PASS 或未来 `5/5` simulation PASS 解释为 live approval。

## 4. 执行顺序

| 顺序 | 任务 | 触发条件 | 产物 |
|---|---|---|---|
| 1 | Phase A r4 runtime authorization | 已完成 | r4 P0-P4 evidence、acceptance check |
| 2 | Phase A r5 runtime authorization | 已完成 | r5 P0-P4 evidence、acceptance check |
| 3 | Phase A r6 runtime authorization | 已完成 | r6 P0-P4 evidence、acceptance check |
| 4 | Stability window summary | 已完成，`completed_runs=5/5` | simulation stability window summary |
| 5 | Phase B readiness closure | 已形成且用户已接受 `READY_WITH_RISK` | readiness closure / release context / runbook addendum / human decision checkpoint |
| 6 | SIM-OPS-GATE-02/03 policy closure | 已完成 | ops policy / ops policy closure check |
| 7 | Phase C CR preparation | 用户明确要准备 small_live/live | live switch CR / decision brief |

## 5. 授权边界

| 操作 | 当前状态 | 规则 |
|---|---|---|
| signed positions readonly | 需要逐次 runtime authorization | 每次执行前确认授权和 gateway identity |
| simulation submit/cancel | 需要逐次 runtime authorization | 每次只执行授权范围内的 runtime，不自动重试 |
| `small_live` / `live` | 未授权 | 必须独立 CR、独立人工决策、独立 runtime authorization |
| gateway 启停 | 用户手工管理 | 如需启动或重启，由用户在 Windows 执行 |
| 修改 Windows runtime 项目 | 条件允许 | 若改动 `/mnt/c/quant-lab-runtime/trading/*`，需要提示用户重启 gateway |
| NAS / provider / lake / catalog / publish / remote Git | 未授权 | 不得触达 |

## 6. 风险与阻断项

| 风险 ID | 状态 | 说明 | 处理 |
|---|---|---|---|
| RISK-STABILITY-WINDOW-INCOMPLETE | closed | simulation runtime 稳定窗口已达到 `5/5` | 进入 Phase B simulation readiness closure |
| RISK-RUNTIME-INPUT-OPERABILITY | policy_defined | r3 已用私有 overlay 通过；输入来源、刷新时机、字段边界和 evidence 脱敏策略已固化 | 若要自动化长期模拟盘，再实现 preflight checker |
| RISK-LIVE-MISREAD | controlled | r3 PASS 可能被误读为 live readiness | 本文档和 STATE 明确 live deferred |
| RISK-AUTHORIZATION-REUSE | controlled | 误复用上一轮 simulation authorization | 每次 runtime 前强制重新确认 |
| RISK-GATEWAY-DESYNC | policy_defined | WSL 仓库和 Windows runtime 项目可能不同步 | 已固化同步、重启和 P0/P0.5 复验规则 |
| RISK-THREE-TRADING-DAY-OBSERVATION | accepted_ready_with_risk | 5/5 count 已完成，但 runbook 保留 3 trading days 观察口径；当前证据主要覆盖两个日期 | 用户已接受 `READY_WITH_RISK`；如未来要求升级 `READY`，可补第三个交易日观察 |

## 7. 下一次恢复流程

下一次恢复本工作时，默认读取顺序：

1. 本文件。
2. `process/state/STATE.current.json`。
3. `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R3-RUNTIME-ACCEPTANCE-2026-06-26.md`。
4. 若准备继续 runtime，再读取 r3 evidence index 和私有 runtime input 路径。
5. 向用户请求新的逐次 `simulation runtime authorization`。
6. 用户启动 Windows gateway 后，从 P0 health / runtime identity 开始。

恢复时禁止：

- 直接跳过 P0。
- 复用 r3 authorization。
- 自动继续 submit/cancel。
- 将 simulation 操作升级为 `small_live` / `live`。

## 8. 当前推荐下一步

`SIM-OPS-GATE-02` 和 `SIM-OPS-GATE-03` 已固化。不要在无新目标下继续重复 runtime submit/cancel。

如果用户希望转向实盘准备，则不要执行 runtime；应先启动独立 live switch CR，并把 `DQ-LIVE-001..004` 作为 CP2 / CP3 前的人工决策输入。
