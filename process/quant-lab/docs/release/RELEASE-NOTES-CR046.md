# CR046 Release Notes

> 状态：`READY_WITH_RISK`，CP8 已人工确认，当前 CR046 framework-first 交付已关闭。
> 范围：framework-first 文档 / 契约 / fixture / 静态验证交付。
> 生成时间：2026-06-18T07:41:38+08:00。

## 交付摘要

CR046 将 QMT 终端策略与 MiniQMT runner 的双目标交付方式收敛为一组框架文档、策略包契约、安装设计、验证框架和后续门禁。它解决的是“后续具体策略应按什么合同交付、如何验证、哪些运行权限必须后置”的问题，不交付具体交易策略，也不证明任何真实运行能力。

## 本次包含

| 类别 | 内容 | 证据 |
|---|---|---|
| 框架 | QMT terminal + MiniQMT runner 双目标策略交付框架 | `process/docs/source-archive/docs/qmt/CR046-DUAL-TARGET-STRATEGY-FRAMEWORK.md` |
| 安装设计 | MiniQMT runner install / uninstall / upgrade / rollback 边界 | `process/docs/source-archive/docs/qmt/CR046-MINIQMT-RUNNER-INSTALL-DESIGN.md` |
| 验证设计 | fixture、静态、文档和后续 runtime 门禁 | `process/docs/source-archive/docs/qmt/CR046-VERIFICATION-FRAMEWORK.md` |
| 研究衔接 | 研究框架反向约束和后续候选路线 | `process/docs/source-archive/docs/research/CR046-RESEARCH-FRAMEWORK-FOLLOWUP.md` |
| 实现证据 | Batch A framework-first 实现完成 | `process/checks/CP6-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-CODING-DONE.md` |
| 验证证据 | CP7 静态 / fixture / 文档 / 契约验证 | `process/checks/CP7-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-VERIFICATION-DONE.md` |

## 不包含

- 不包含具体策略包、真实策略导入或终端运行。
- 不包含 QMT/MiniQMT/XtQuant/gateway 启动、连接或安装。
- 不包含 NAS 访问、`.env` / 凭据 / 账号 / 账户 / 资金 / 持仓 / 委托 / 成交 / 日志原文读取。
- 不包含 submit/cancel、simulation/live、provider fetch、lake write 或 catalog publish。
- 不自动启动 CR089 或 CR091。

## 质量结论

| 检查 | 结论 | 说明 |
|---|---|---|
| CP6 | `PASS` | framework-first 文档 / 契约实现完成。 |
| CP7 | `PASS_WITH_RISK` | 验证只覆盖静态、fixture、文档和契约。 |
| CP8 推荐 | `READY_WITH_RISK` | 剩余风险可追踪，需用户在 CP8 接受。 |

## 已知风险

| 风险 | 等级 | CP8 处理 |
|---|---|---|
| `R-CR046-CP7-001` docs/product canonical scenarios/test matrix 缺失 | medium | 接受为后续文档治理风险，不在 CR046 内静默迁移。 |
| `R-CR046-CP7-002` CR-INDEX 全量 YAML parse 历史债务 | medium | 接受为账本卫生风险，CR046 局部追踪继续以一致性脚本为准。 |
| `R-CR046-CP7-003` 无真实 runtime evidence | high | 接受为本轮 scope boundary；不声明 runtime verified。 |
| `REV-CR046-004` 历史 docs/qmt 路径引用 | low | 改以 source-archive 路径追溯，不阻断。 |

## 后续路线

| 路线 | 状态 | 说明 |
|---|---|---|
| CR047-candidate | candidate | 首个具体研究策略双目标交付。 |
| CR048-candidate | candidate | QMT terminal shadow / submit-cancel 最小验证，需运行授权。 |
| CR049-candidate | candidate | MiniQMT/XtQuant 只读连接与 runner install 实机验证。 |
| CR050-candidate | spike_candidate | tick 级 runner / 资源控制 Spike。 |
| CR091 | blocked | runner 后续研究 / 设计 / 实现路线，不由 CR046 自动启动。 |

## 发布决策

用户已在 2026-06-18T07:59:20+08:00 同意 CP8，CR046 当前 framework-first 交付按 `READY_WITH_RISK` 关闭。关闭后，CR046 不再阻塞后续候选登记，但任何真实运行、连接、交易、NAS 或敏感数据动作仍必须另起独立 CR 和授权门禁。
