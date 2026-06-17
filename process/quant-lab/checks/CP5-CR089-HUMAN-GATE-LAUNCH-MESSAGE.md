请审查：process/checkpoints/CP5-CR089-QMT-INTERFACE-VALIDATION-READINESS.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP5-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml（read_profile=compact，status=ready）
上下文胶囊：process/context/CP5-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：7
决策收集覆盖：已扫描 STATE、CP2/CP3 checkpoint、CP5 自动预检、策略包交付规划和离线测试证据；候选问题 29 项，纳入待决策 7 项；discussion log N/A。
如果你回复 approve，表示你接受以下 7 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR089-01 | implementation | 后续是否允许准备本地离线策略包骨架？ | 允许仅本地生成/整理 qmt_interface_smoke package skeleton 和 manifest/checksum 说明。 | 只保留文档；直接发布 NAS。 | 推荐可验证合同。 | 不触碰 NAS。 |
| DQ-CP5-CR089-02 | runtime_authorization | 是否授权 agent 运行 QMT/MiniQMT？ | 不授权；真实接口由用户手工运行。 | agent 代跑；远程 trading_pc 执行。 | 推荐最小权限。 | 自动化较低。 |
| DQ-CP5-CR089-03 | runtime_authorization | 是否授权 NAS 操作？ | 不授权 NAS publish/pull/list/copy/delete。 | 授权 publish；pull；listing。 | 推荐不接触外部存储。 | NAS 实际状态未知。 |
| DQ-CP5-CR089-04 | security | smoke 证据如何回填？ | 只回填 run_id、endpoint_id、status、position_count、auth_status、redaction_status。 | 粘贴原始 positions；上传日志。 | 推荐保护账户隐私。 | 诊断性较弱。 |
| DQ-CP5-CR089-05 | risk_acceptance | endpoint matrix 冲突如何处理？ | 作为后续实现前风险，不在本门禁改代码。 | 立即修复；忽略。 | 推荐避免破坏 CR020 特例。 | 后续实现需重审。 |
| DQ-CP5-CR089-06 | security | 是否允许 submit/cancel/simulation/live？ | 全部不允许。 | simulation；小额 live；cancel dry-run。 | 推荐保护交易安全。 | 不验证写接口。 |
| DQ-CP5-CR089-07 | follow_up_tracking | CP5 approve 后 CR089 状态如何处理？ | 保持 blocked-readiness-approved，等待 CR046 active lock 或后续 runtime authorization。 | 改 active；关闭；合并 CR046。 | 推荐避免双 active。 | 不能自动进入 runtime。 |
不授权项：
- 不授权 agent 启动 QMT、MiniQMT、XtQuant、gateway 或远程交易主机命令。
- 不授权读取 .env、token、账号、密码、HMAC secret、cookie、session、private key 或 QMT 凭据。
- 不授权访问、列目录、读取、写入、复制、发布、拉取或删除 NAS 内容。
- 不授权账户原文查询、持仓原文粘贴、原始日志上传。
- 不授权 submit_order、cancel_order、simulation、live_readonly、small_live、scale_up。
- 不授权修改 endpoint matrix / query_positions 语义冲突。
- 不授权把 CR089 设为 active 或恢复 CR020/CR046。
approve
修改: <具体修改点>
reject
