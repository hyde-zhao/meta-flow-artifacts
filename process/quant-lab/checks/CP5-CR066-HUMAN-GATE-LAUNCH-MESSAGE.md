请审查：process/checkpoints/CP5-CR066-CLEAN-PUBLICATION-READINESS.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP5-CR066-CLEAN-PUBLICATION-READINESS-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 32 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权以下 14 项禁止操作。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR066-01 | runtime_authorization | CR066 approve 是否授权 Git 写动作？ | 不授权；只批准策略，执行另起 CR067。 | clean branch；commit docs；remote add/push。 | 推荐最安全。 | publication 仍暂停。 |
| DQ-CP5-CR066-02 | implementation | clean manifest policy？ | allowlist / default exclude / conditional include。 | 全仓；code-only；current branch as-is。 | 推荐兼顾安全和可用。 | 后续需扫描。 |
| DQ-CP5-CR066-03 | security | 后续 execution scan 是否硬门？ | no-secret/data/topology/large-file scan 必须执行。 | 只 secret scan；只记录风险。 | 推荐覆盖完整。 | 可能阻断 CR067。 |
| DQ-CP5-CR066-04 | risk_acceptance | 是否接受 publication 延迟？ | 接受，换取 clean baseline。 | as-is 风险接受；取消 publication。 | 推荐保护公共发布面。 | 延迟远端发布。 |
| DQ-CP5-CR066-05 | follow_up_tracking | 是否创建 CR067 执行门禁？ | 是，单独处理 clean snapshot/branch/remote/push 授权。 | 回 CR062；转 NAS/lake/runtime；停止。 | 推荐边界清晰。 | 增加后续 CR。 |

不授权项：
- NA-CP5-CR066-001 不执行 `git add`。
- NA-CP5-CR066-002 不执行 `git commit`。
- NA-CP5-CR066-003 不执行 `git remote add` / `git remote set-url`。
- NA-CP5-CR066-004 不执行 `git push`。
- NA-CP5-CR066-005 不执行 `git tag`。
- NA-CP5-CR066-006 不执行 branch creation / branch rename / default branch rename。
- NA-CP5-CR066-007 不执行 history rewrite / rebase-publication / force push。
- NA-CP5-CR066-008 不删除 remote 或远端 ref。
- NA-CP5-CR066-009 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。
- NA-CP5-CR066-010 不执行 NAS 数据 copy / move / delete / archive promote。
- NA-CP5-CR066-011 不执行 `MARKET_DATA_LAKE_ROOT` replacement、provider fetch、lake write、catalog publish。
- NA-CP5-CR066-012 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。
- NA-CP5-CR066-013 不恢复或推进 CR046 CP7。
- NA-CP5-CR066-014 不执行 physical move / rename / delete 或 bulk import rewrite。

回复：
approve
修改: <具体修改点>
reject
