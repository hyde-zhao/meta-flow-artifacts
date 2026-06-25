---
phase_goal_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-PHASE-GOALS-2026-06-25"
created_at: "2026-06-25T10:55:00+08:00"
created_by: "host-orchestrator"
status: "phase-goals-recorded"
workflow_mode: "standard"
current_baseline_status: "simulation-smoke-passed"
primary_next_target: "simulation-multifactor-strategy-runtime-ready"
deferred_target: "simulation-to-live-switch-readiness"
deferred_target_status: "deferred-not-implemented"
implementation_allowed: false
runtime_authorization_granted: false
approval_impact_reference: true
source_refs:
  - "process/context/RUNNER-SIMULATION-TRADING-MACHINE-SMOKE-HANDOFF-2026-06-24.md"
  - "process/checks/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-VERIFICATION-2026-06-24.md"
  - "process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json"
---

# Runner QMT Simulation Multifactor Strategy Runtime Phase Goals

## 1. 放置位置与用途

本文档放在 `process/context/`，作为下一阶段目标胶囊，而不是正式需求基线、HLD、LLD 或实现计划。

原因：

- 当前 `RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-2026-06-24` 已完成，状态为 `simulation-smoke-passed`。
- 用户的最终阶段目标是推进到“可以在 QMT 模拟盘运行多因子策略”。
- “模拟盘到实盘切换”需要记录为后续阶段目标，但暂时不实施；它需要单独讨论切换方式、切换条件、验收标准、授权边界和风险接受。
- 后续 CP2 / CP3 / CP5 / CP8 或 runtime authorization 人工审批时，如审批内容涉及 runtime profile、MiniQMT 账户类型、gateway mode、策略自动下单、实盘切换或 live/small_live，应读取本文档并分析对阶段目标的影响。

本文档不授权任何新增 runtime 操作，不授权 `small_live` / `live` submit/cancel，不授权读取或保存凭据、账号原文、证券代码原文、原始订单引用或资金明细。

## 2. 已完成基线

| 项目 | 状态 | 证据 |
|---|---|---|
| QMT gateway health | PASS | `session_ready=true`、`runtime_status=xtquant-ready` |
| positions readonly | PASS | 仅保存脱敏摘要 |
| simulation buy submit/cancel | PASS | submit accepted，cancel accepted |
| simulation sell submit/cancel | PASS | submit accepted，cancel accepted |
| `cancel-ref` 撤单链路 | PASS | client 不接触原始 broker order ref |
| 脱敏 evidence | PASS | 未保存凭据、账号原文、证券代码原文、原始订单引用或资金明细 |
| `small_live` / `live` | NOT_AUTHORIZED | 当前未实施、未验证 |

## 3. 阶段目标总览

| 阶段 | 目标 | 实施状态 | 是否需要人工审批 | 说明 |
|---|---|---|---|---|
| P0 | 固化 simulation runtime profile 与安全身份识别 | 待启动 | 是 | 防止模拟盘 runner 误连实盘 gateway |
| P1 | 多因子策略生成目标组合 | 待启动 | 是 | 从信号生成 target portfolio，不下单 |
| P2 | 目标组合生成 simulation order plan | 待启动 | 是 | current positions -> target portfolio -> buy/sell intents |
| P3 | simulation execution engine | 待启动 | 是 | 批量 submit/cancel、OMS 状态、节流、异常停止 |
| P4 | simulation reconciliation 与运行报告 | 待启动 | 是 | 订单、持仓、目标组合对账，只保存脱敏摘要 |
| P5 | 模拟盘长期运行 runbook / 运营门禁 | 待启动 | 是 | 盘前、盘中、盘后、异常恢复和人工接管 |
| P6 | 模拟盘到实盘切换场景与需求讨论 | 暂缓 | 是 | 只记录目标；等模拟盘完成后再讨论 |
| P7 | small_live/live 实现与验证 | 暂缓 | 是，高风险 | 依赖 P6 讨论和独立授权，不属于当前阶段 |

## 4. 近期主目标：模拟盘运行多因子策略

目标状态：`simulation-multifactor-strategy-runtime-ready`

范围：

- 仅使用模拟 MiniQMT 账户启动的 QMT gateway。
- runner 必须确认 gateway runtime mode/profile 是 simulation。
- 多因子策略生成目标组合。
- 目标组合转换为 buy/sell order intents。
- order intents 通过 stage gate、pretrade risk、kill switch 和 reconciliation precheck 后，才允许 simulation submit。
- 支持 simulation buy/sell submit/cancel。
- 支持 cancel-all current run。
- 支持运行后 reconciliation。
- 只保存脱敏 evidence。

非范围：

- 不实施 `small_live` / `live` submit/cancel。
- 不实现模拟盘到实盘自动切换。
- 不保存或推送凭据、账号原文、证券代码原文、原始订单引用或资金明细。
- 不把 simulation runtime authorization 复用为 live authorization。

## 5. Simulation Runtime Profile 目标

下一阶段应先把 runtime profile 显式化，避免靠推断判断“模拟盘还是实盘”。

建议新增或固化的字段：

| 字段 | 示例 | 说明 |
|---|---|---|
| `QMT_RUNTIME_MODE` | `simulation` | gateway 启动时固定 runtime mode |
| `QMT_ACCOUNT_KIND` | `simulation` | 用户声明当前 MiniQMT 登录账户类型 |
| `QMT_RUNTIME_PROFILE` | `cr138-simulation` | 脱敏 profile 名称，用于 client/gateway handshake |
| `QMT_RUNTIME_AUTHORIZATION_REF` | `runner-qmt-simulation-...` | 单次运行授权引用 |
| `expected_runtime_mode` | `simulation` | client 请求侧期望 |
| `expected_runtime_profile` | `cr138-simulation` | client 请求侧期望 |

必须 fail closed 的情况：

- simulation runner 连接到 `runtime_mode=live` 的 gateway。
- live runner 连接到 `runtime_mode=simulation` 的 gateway。
- gateway 未声明 `QMT_ACCOUNT_KIND`。
- request expected runtime 与 gateway identity 不匹配。
- live scope 请求打到 simulation gateway。
- simulation scope 请求打到 live gateway。

## 6. 多因子模拟盘运行阶段

### P1：多因子目标组合

目标：

- 从多因子信号生成 target portfolio。
- 输出可回放、可审计、可解释的目标组合摘要。
- 不触达 QMT submit/cancel。

建议第一版：

- topN 打分组合。
- 等权或上限权重。
- 股票池白名单或既有策略 universe。
- 输出稳定 `instrument_ref`，不在 evidence 保存原始证券代码。

退出条件：

- target portfolio 可生成。
- 空组合、过期数据、异常因子值可被阻断。
- 输出 evidence 不含原始标的、账号或凭据。

### P2：Order Plan

目标：

- 从 current positions 与 target portfolio 生成 buy/sell order intents。
- 先生成计划，不直接提交。

必须覆盖：

- 买入意图。
- 卖出意图。
- 非 100 股整数倍处理。
- 可卖数量不足时阻断或裁剪。
- 现金不足时阻断或裁剪。
- 目标权重、单票上限、换手上限。

退出条件：

- order plan 可审计。
- 风控失败时不触达 gateway。
- evidence 只保存数量级、计数、digest 和脱敏 refs。

### P3：Simulation Execution Engine

目标：

- 对通过风控的 order intents 执行 simulation submit/cancel。
- 支持批量、节流、异常停止和 cancel-all。

必须覆盖：

- buy submit/cancel。
- sell submit/cancel。
- submit accepted 后返回 `cancel-ref`。
- cancel 使用 `cancel-ref`。
- submit accepted 但 cancel failed 时停止后续 submit，并要求人工 MiniQMT 检查。
- 任意未知订单状态进入 unresolved，阻断下一轮自动运行。

退出条件：

- fake adapter / contract test 通过。
- 交易机 simulation runtime smoke 通过。
- 原始 broker order ref 不输出给 client 或 evidence。

### P4：Reconciliation

目标：

- 对运行前后持仓、订单状态、目标组合偏差做对账。
- 为后续长期运行提供恢复点。

必须覆盖：

- pre-run positions snapshot digest。
- post-run positions snapshot digest。
- order intent count、submitted count、cancelled count、rejected count、unknown count。
- drift bucket。
- unresolved orders 阻断下一轮。

退出条件：

- reconciliation result 可解释。
- unresolved 为 0 或进入人工接管队列。
- evidence 不包含资金、账号、标的、订单原文。

### P5：Operational Runbook

目标：

- 让用户能够按固定步骤运行模拟盘多因子策略。

必须覆盖：

- 盘前检查。
- gateway health。
- runtime profile 检查。
- target portfolio 生成。
- order plan 人工确认或自动阈值。
- simulation submit/cancel。
- reconciliation。
- 异常恢复。
- 日终取消未完成订单。

退出条件：

- runbook 可执行。
- 运行失败路径有明确人工接管动作。
- process evidence 与 run ledger 可追溯。

## 7. 暂缓目标：模拟盘到实盘切换

目标 ID：`DEFERRED-SIMULATION-TO-LIVE-SWITCH`

状态：`deferred-not-implemented`

记录原因：

- 用户要求将模拟盘到实盘切换纳入阶段目标，但后续再讨论具体场景和需求。
- 当前重点是先完成模拟盘多因子策略运行闭环。
- 实盘切换涉及真实资金、账户类型、授权、限额、回滚、人工审批和运行事故处理，不能从 simulation smoke 直接外推。

后续必须讨论的问题：

| ID | 问题 | 需要决策 |
|---|---|---|
| DQ-LIVE-001 | 切换方式 | 独立 live gateway profile、独立端口、独立 env，还是同 gateway 多 profile；推荐独立 profile |
| DQ-LIVE-002 | 切换触发 | 手动审批、固定日期、连续模拟盘稳定运行 N 日，或组合条件 |
| DQ-LIVE-003 | 切换标准 | simulation 运行成功率、reconciliation 通过率、无 unresolved orders、风控命中率 |
| DQ-LIVE-004 | 实盘范围 | small_live 还是 full live；初始标的、方向、数量、金额上限 |
| DQ-LIVE-005 | 授权方式 | per-run authorization、时间窗口、授权人、可撤单授权 |
| DQ-LIVE-006 | 回滚方式 | kill switch、cancel-only、停止新单、人工 MiniQMT 接管 |
| DQ-LIVE-007 | 证据边界 | 哪些 live 结果可保存为脱敏摘要，哪些原文禁止保存 |
| DQ-LIVE-008 | 审批门 | CP2/CP3/CP5/CP8 是否需要重新打开，或作为新目标包全流程推进 |

后续建议切换条件草案：

- 模拟盘多因子策略连续运行达到用户确认的稳定窗口。
- 每次运行 reconciliation 通过。
- unresolved orders 为 0。
- cancel-all current run 已验证。
- runtime profile handshake 已实现并测试。
- live-readonly 脱敏验证通过。
- small_live endpoint、scope、限额、白名单、kill switch 和 cancel policy 已实现并测试。
- 用户完成独立高风险人工审批。

暂缓期间禁止：

- 实现 live submit/cancel runtime route。
- 打开 live submit/cancel client transport。
- 使用真实 MiniQMT 账户执行策略 submit/cancel。
- 将 simulation 授权复用为 live 授权。
- 在 evidence 中保存真实账号、资金、标的或订单原文。

## 8. 后续人工审批影响分析要求

任何后续人工审批，如果命中以下任一条件，必须读取本文档并在 Decision Brief 中增加“阶段目标影响分析”：

- 修改模拟盘多因子策略运行目标。
- 修改 runtime profile / account kind / gateway mode。
- 允许策略自动 submit/cancel。
- 新增或修改 live / small_live endpoint。
- 新增或修改 live-readonly。
- 修改 HMAC scope 或 runtime authorization 语义。
- 修改限额、白名单、kill switch、cancel policy 或 reconciliation gate。
- 将模拟盘运行结果作为进入实盘的依据。

影响分析至少回答：

| 维度 | 必答问题 |
|---|---|
| 范围 | 本审批是否改变 P0-P5 近期模拟盘目标？ |
| 暂缓项 | 是否触发 `DEFERRED-SIMULATION-TO-LIVE-SWITCH`？ |
| 授权 | 是否新增 runtime / trading / credential / account / live 权限？ |
| 安全 | 是否可能让 simulation runner 误连 live gateway？ |
| 证据 | 是否仍满足不保存原文凭据、账号、标的、订单和资金？ |
| 回滚 | 失败后是否能停止新单、撤单或人工接管？ |
| 状态 | 是否需要新建 CR / Change Package，或重新打开 CP2 / CP3 / CP5 / CP8？ |

## 9. 推荐下一步

推荐启动新目标包：

`RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25`

推荐第一阶段只做：

1. runtime profile / account kind 显式化。
2. simulation runner 验证 gateway profile。
3. 多因子 target portfolio 生成。
4. order plan 与 pretrade risk。
5. 不进入 `small_live` / `live`。

`DEFERRED-SIMULATION-TO-LIVE-SWITCH` 作为后续目标保留，等模拟盘多因子策略运行闭环完成后，再进入场景发现、需求澄清和人工审批。
