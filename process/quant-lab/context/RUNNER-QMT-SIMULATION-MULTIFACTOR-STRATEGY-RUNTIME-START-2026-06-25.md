---
start_context_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-START-2026-06-25"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T11:21:41+08:00"
created_by: "host-orchestrator"
status: "started"
workflow_mode: "standard"
current_baseline_status: "simulation-smoke-passed"
target_status: "simulation-multifactor-strategy-runtime-ready"
current_phase_goal: "P0-runtime-profile-and-safety-identity"
implementation_allowed: false
runtime_authorization_granted: false
live_authorization_granted: false
source_phase_goals: "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-PHASE-GOALS-2026-06-25.md"
upstream_smoke_handoff: "process/context/RUNNER-SIMULATION-TRADING-MACHINE-SMOKE-HANDOFF-2026-06-24.md"
upstream_smoke_verification: "process/checks/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-VERIFICATION-2026-06-24.md"
upstream_smoke_evidence: "process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json"
---

# Runner QMT Simulation Multifactor Strategy Runtime Start

## 1. 启动结论

`RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25` 已启动为下一阶段目标包。

本启动只更新运行态、交接上下文和 ledger，不执行任何 QMT gateway、MiniQMT、账户查询、submit/cancel、provider/lake/catalog、NAS 或远端写入动作。

## 2. 当前基线

| 项目 | 状态 | 证据 |
|---|---|---|
| 上游模拟盘 smoke | PASS | `process/checks/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-VERIFICATION-2026-06-24.md` |
| buy simulation submit/cancel | PASS | `process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json` |
| sell simulation submit/cancel | PASS | `process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json` |
| 脱敏边界 | PASS | 未保存凭据、账号原文、证券代码原文、原始订单引用或资金明细 |
| small_live / live | NOT_AUTHORIZED | 当前阶段目标明确暂缓 |

## 3. 本目标包范围

目标状态：`simulation-multifactor-strategy-runtime-ready`。

本目标包按 `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-PHASE-GOALS-2026-06-25.md` 启动，当前只进入 P0：

| 阶段 | 状态 | 下一步 |
|---|---|---|
| P0 runtime profile 与安全身份识别 | active-next | 固化 `QMT_RUNTIME_MODE=simulation`、`QMT_ACCOUNT_KIND=simulation`、`QMT_RUNTIME_PROFILE`、`expected_runtime_mode`、`expected_runtime_profile` 的 fail-closed 合同 |
| P1 多因子目标组合 | pending | P0 合同确认后再进入 |
| P2 simulation order plan | pending | P1 输出 target portfolio 后再进入 |
| P3 simulation execution engine | pending | P2 风控、kill switch 和 reconciliation precheck 就绪后再进入 |
| P4 reconciliation 与运行报告 | pending | P3 受控执行后再进入 |
| P5 operational runbook | pending | P0-P4 闭环后再进入 |
| P6 模拟盘到实盘切换讨论 | deferred | 仅保留目标，不实施 |
| P7 small_live/live 实现与验证 | deferred-not-authorized | 需要独立高风险授权门 |

## 4. 不授权边界

本启动不授权：

- 真实资金 `small_live` / `live` submit/cancel。
- 读取、保存、打印或推送凭据、token、secret、账号原文、证券代码原文、原始订单引用或资金明细。
- 直接触发 QMT gateway runtime、账户查询、submit/cancel 或 cancel-all。
- 真实 NAS 内容操作、provider/lake/catalog/publish、远端 Git 写入。
- 将 simulation runtime authorization 复用为 live authorization。

## 5. 下一步

下一步应先形成 P0 的最小设计 / 实现授权边界：

1. 定义 runtime profile handshake 的字段、来源、默认值和 fail-closed 条件。
2. 明确 simulation runner 与 gateway identity mismatch 的失败路径。
3. 形成最小实现对象与验证清单，但在未获得后续明确授权前不触发真实 runtime。
4. 将 P6/P7 的 live 切换事项保留为 deferred，不进入当前实现。

## 6. 启动状态

| 字段 | 值 |
|---|---|
| workflow_id | `RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25` |
| current_phase_goal | `P0-runtime-profile-and-safety-identity` |
| status | `started` |
| implementation_allowed | `false` |
| runtime_authorization_granted | `false` |
| live_authorization_granted | `false` |
