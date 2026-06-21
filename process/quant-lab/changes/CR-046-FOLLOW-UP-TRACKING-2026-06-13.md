---
tracking_id: "CR-046-FOLLOW-UP-TRACKING"
source_cr: "CR-046"
status: "closed-current-delivery"
created_at: "2026-06-13T21:46:39+08:00"
updated_at: "2026-06-19T19:35:00+08:00"
owner: "meta-po"
index_path: "process/changes/CR-INDEX.yaml"
---

# CR-046 后续事项台账

## 范围说明

本台账只登记 CR046 framework-first 后续候选项，不预创建正式 CR 文件。候选项启动前必须重新执行冲突预检、CP2 intake、权限边界确认和对应检查点；候选项不占 active formal CR 锁。

CR046 当前只冻结早期 framework-first 策略交付框架、验证框架设计和策略包契约。不授权具体策略交付、QMT 终端运行验证、MiniQMT 连接、账户查询、submit/cancel、simulation/live、provider fetch、lake write 或 catalog publish。

挂起记录：用户于 2026-06-14T00:29:41+08:00 要求 CR046 先挂起；恢复点为 CP6 PASS / ready-for-verification。用户于 2026-06-18 明确恢复 CR046 并仅允许 CP7 静态 / fixture / 文档 / 契约验证；CP7 已 `PASS_WITH_RISK`。用户于 2026-06-18T07:59:20+08:00 同意 CP8，CR046 当前 framework-first 交付已关闭为 `closed-current-delivery / READY_WITH_RISK`。候选项仍不得自动启动，不得执行真实传输、导入、运行、连接或交易。

2026-06-19 架构重对齐：用户明确 runner 属于 quant-lab，不属于 MiniQMT；MiniQMT 只是当前接口适配目标，后续可接掘金量化等接口。策略交付也必须按跨平台 target 模型设计，当前仅实现 QMT direct-run target。因此，早期“QMT / MiniQMT 双目标”和“MiniQMT runner install”表述保留为历史基线，不再作为当前主线；当前主线应先通过 `CR091-FU-05` 重对齐 target / adapter taxonomy。

## 后续候选项

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR047-candidate | 首个跨平台策略交付（当前仅 QMT target） | candidate-reframed | CR | 2 |  | blocked_by=CR091-FU-05 | strategy_delivery_gate | 旧“双目标交付”表述需先由 CR091-FU-05 重对齐；仍需用户选择具体策略并重新做 CP2/CP3/CP5 | 等 target taxonomy / manifest schema / QMT entrypoint contract 稳定后，选择一个已通过研究准入的策略做 QMT direct-run 交付 |
| CR048-candidate | QMT direct-run validation gate | candidate-reframed | CR | 3 |  | blocked_by=CR047 / CR091-FU-05 | runtime_authorization | 需具体策略包、QMT direct-run entrypoint 和脱敏 evidence schema | 用户逐 run 授权后由用户在 QMT 环境手工验证；agent 不代跑、不读账号 / 日志原文 |
| CR049-candidate | MiniQMT gateway adapter readonly validation | candidate-reframed | CR / Spike | 4 |  | blocked_by=CR091-FU-05 / MiniQMT 权限 | environment_authorization | MiniQMT 仅作为 API gateway adapter，不作为 runner；用户当前未授权 agent 连接 | 权限和逐 run 授权具备后验证 gateway health / capabilities / query_positions，只回填脱敏 evidence |
| CR050-candidate | Future adapter / resource-control Spike | spike_candidate-reframed | Spike | 5 |  | blocked_by=adapter protocol stability | research_spike | tick 级、其他接口、资源控制和事件驱动复杂度高，不属于当前 QMT-only 策略交付 | QMT direct-run 与 MiniQMT gateway adapter 稳定后再评估掘金量化 / tick / 其他接口 |
| CR051 | 策略研究生命周期框架与策略类型 taxonomy | closed-current-delivery | CR | 1 | `process/changes/CR-051-STRATEGY-RESEARCH-LIFECYCLE-FRAMEWORK-2026-06-14.md` | consumed-and-closed | research_lifecycle_framework_gate | 已关闭当前交付；不占 CR046 CP8 阻塞位 | 保留审计证据；后续研究 runner 或策略生命周期扩展需独立 CR |

## 不授权边界

| 项 | 状态 | 说明 |
|---|---|---|
| 具体策略交付 | not-authorized | CR091-FU-05 / CR047 启动后重新确认；当前只规划 QMT direct-run target |
| QMT 终端 shadow / 模拟盘运行验证 | not-authorized | CR048 或等价 runtime gate；必须用户逐 run 授权 |
| MiniQMT / XtQuant 连接 | not-authorized | CR049 或等价环境授权；MiniQMT 只作为 API gateway adapter，不作为策略 runner |
| 账户 / 资金 / 持仓 / 委托 / 成交查询 | not-authorized | 需只读连接门禁 |
| submit / cancel / simulation / live | not-authorized | 需高风险运行授权 |
| provider fetch / lake write / catalog publish | not-authorized | 不属于 CR046 |

## 状态同步

| 对象 | 路径 | 状态 |
|---|---|---|
| 正式 CR | `process/changes/CR-046-TERMINAL-NATIVE-SIMULATION-STRATEGY-EXPORT-2026-06-13.md` | closed-current-delivery / READY_WITH_RISK |
| CR 索引 | `process/changes/CR-INDEX.yaml` | no-active-formal-cr；后续候选已改为 candidate-reframed |
| STATE | `process/STATE.md` | active_change=''; blocked=false；next_gate=用户选择后续候选；pending_gate='' |
