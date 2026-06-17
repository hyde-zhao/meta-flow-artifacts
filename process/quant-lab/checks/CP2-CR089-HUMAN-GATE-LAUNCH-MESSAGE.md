请审查：process/checkpoints/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP2-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml（read_profile=compact，status=ready）
上下文胶囊：process/context/CP2-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：5
决策收集覆盖：已扫描 STATE、CR089 formal CR、启动冲突预检、策略包交付规划和用户显式选择；候选问题 25 项，纳入待决策 5 项；discussion log N/A。
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR089-01 | scope | 是否进入 CR089 门禁材料阶段？ | 进入 CP2/CP3/CP5 材料阶段，CR089 保持 blocked。 | 合并 CR046；等待 CR046 CP8；取消。 | 推荐保留路线且不破坏 active lock。 | CR046 仍持有 active lock。 |
| DQ-CP2-CR089-02 | scope | 本轮范围是否仅覆盖策略包 + 只读 smoke？ | 仅 package exchange、manifest、trading_pc intake 和 query_positions smoke。 | health/capabilities；submit/cancel；simulation。 | 推荐权限最小。 | 不覆盖交易写接口。 |
| DQ-CP2-CR089-03 | security | CR020 是否恢复？ | 不恢复，仅审计复用只读接口代码和 runbook 边界。 | 恢复 CR020；复制旧根 STATE。 | 推荐符合 deleted-by-user。 | 防止旧状态污染。 |
| DQ-CP2-CR089-04 | runtime_authorization | CP2 是否授权 NAS/QMT/凭据/账户动作？ | 不授权，仅确认范围。 | CP2 直接授权 NAS pull；agent 代跑 QMT。 | 推荐最小权限。 | 无外部副作用。 |
| DQ-CP2-CR089-05 | follow_up_tracking | CR089 和 CR046 如何登记？ | CR089 继续 blocked formal CR；CR046 保持 active。 | CR089 改 active；关闭或恢复 CR046。 | 推荐避免双 active。 | CR089 不会自动执行。 |
不授权项：
- 不授权访问、列目录、读取、写入、复制或删除 NAS 内容。
- 不授权读取 .env、token、API key、账号、密码、HMAC secret、cookie、session、private key 或 QMT 凭据。
- 不授权启动 QMT、MiniQMT、XtQuant、gateway 或任何 broker runtime。
- 不授权账户原文查询、持仓原文输出、成交 / 委托原文输出。
- 不授权 submit_order、cancel_order、simulation、live_readonly、small_live 或 scale_up。
- 不授权恢复 CR020、从旧根 STATE 恢复状态、覆盖当前 CR046 active lock。
approve
修改: <具体修改点>
reject
