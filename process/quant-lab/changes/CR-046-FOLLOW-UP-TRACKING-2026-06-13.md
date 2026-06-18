---
tracking_id: "CR-046-FOLLOW-UP-TRACKING"
source_cr: "CR-046"
status: "closed-current-delivery"
created_at: "2026-06-13T21:46:39+08:00"
updated_at: "2026-06-18T07:59:20+08:00"
owner: "meta-po"
index_path: "process/changes/CR-INDEX.yaml"
---

# CR-046 后续事项台账

## 范围说明

本台账只登记 CR046 framework-first 后续候选项，不预创建正式 CR 文件。候选项启动前必须重新执行冲突预检、CP2 intake、权限边界确认和对应检查点；候选项不占 active formal CR 锁。

CR046 当前只冻结 QMT / MiniQMT 双目标策略交付框架、验证框架设计、MiniQMT runner 安装设计和策略包契约。不授权具体策略交付、QMT 终端运行验证、MiniQMT 连接、账户查询、submit/cancel、simulation/live、provider fetch、lake write 或 catalog publish。

挂起记录：用户于 2026-06-14T00:29:41+08:00 要求 CR046 先挂起；恢复点为 CP6 PASS / ready-for-verification。用户于 2026-06-18 明确恢复 CR046 并仅允许 CP7 静态 / fixture / 文档 / 契约验证；CP7 已 `PASS_WITH_RISK`。用户于 2026-06-18T07:59:20+08:00 同意 CP8，CR046 当前 framework-first 交付已关闭为 `closed-current-delivery / READY_WITH_RISK`。候选项仍不得自动启动，不得执行真实传输、导入、运行、连接或交易。

## 后续候选项

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR047-candidate | 首个研究策略双目标交付 | candidate | CR | 1 |  | requires_user_start | strategy_delivery_gate | CR046 框架已 CP8 关闭；仍需用户选择具体策略并重新做 CP2/CP3/CP5 | 选择一个已通过研究准入的策略进行 QMT / MiniQMT 双目标交付 |
| CR048-candidate | QMT 终端真实模拟盘最小 submit/cancel 授权验证 | candidate | CR | 2 |  | blocked_by=CR047 | runtime_authorization | 需具体策略包和 QMT shadow 证据 | 用户逐 run 授权后启动 |
| CR049-candidate | MiniQMT / XtQuant 只读连接与 runner install 实机验证 | candidate | CR / Spike | 3 |  | blocked_by=MiniQMT 权限 | environment_authorization | 用户当前没有 MiniQMT 权限 | 权限开通后验证 userdata_mini、xtquant、账号只读、runner install |
| CR050-candidate | tick 级 MiniQMT runner / 资源控制 Spike | spike_candidate | Spike | 5 |  | blocked_by=MiniQMT 只读和日频模拟盘稳定证据 | research_spike | tick 级复杂度和资源控制高，不属于本轮 | 日频双目标稳定后再评估 |
| CR051 | 策略研究生命周期框架与策略类型 taxonomy | closed-current-delivery | CR | 1 | `process/changes/CR-051-STRATEGY-RESEARCH-LIFECYCLE-FRAMEWORK-2026-06-14.md` | consumed-and-closed | research_lifecycle_framework_gate | 已关闭当前交付；不占 CR046 CP8 阻塞位 | 保留审计证据；后续研究 runner 或策略生命周期扩展需独立 CR |

## 不授权边界

| 项 | 状态 | 说明 |
|---|---|---|
| 具体策略交付 | not-authorized | CR047 启动后重新确认 |
| QMT 终端 shadow / 模拟盘运行验证 | not-authorized | CR048 或等价 runtime gate |
| MiniQMT / XtQuant 连接 | not-authorized | CR049 或等价环境授权 |
| 账户 / 资金 / 持仓 / 委托 / 成交查询 | not-authorized | 需只读连接门禁 |
| submit / cancel / simulation / live | not-authorized | 需高风险运行授权 |
| provider fetch / lake write / catalog publish | not-authorized | 不属于 CR046 |

## 状态同步

| 对象 | 路径 | 状态 |
|---|---|---|
| 正式 CR | `process/changes/CR-046-TERMINAL-NATIVE-SIMULATION-STRATEGY-EXPORT-2026-06-13.md` | closed-current-delivery / READY_WITH_RISK |
| CR 索引 | `process/changes/CR-INDEX.yaml` | active-formal-cr |
| STATE | `process/STATE.md` | active_change=''; blocked=false；next_gate=用户选择后续候选；pending_gate='' |
