请审查：`process/checkpoints/CP8-CR060-DELIVERY-READINESS.md`

自动预检结论：PASS。CR060 docs-only identity convergence 已完成，建议 `READY_WITH_RISK` 关闭当前交付。

Context Capsule 摘要：`process/context/CP8-CR060-DELIVERY-CONTEXT.yaml`，read_profile=compact；本轮只包含 README / USER-MANUAL 身份收敛，不包含 Git remote、NAS、data lake、runtime 或 CR046。

决策收集覆盖摘要：已扫描 6 个来源，发现候选问题 5 个，纳入待决策 5 个，N/A 0。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，并关闭 CR060 当前 docs-only delivery；不表示授权以下 8 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR060-01 | risk_acceptance | 是否接受 CR060 READY_WITH_RISK？ | 接受并关闭 CR060 当前 docs-only delivery。 | reject 并回滚文档改写。 | 推荐方案完成第一真实迁移切片；回滚会退回旧身份。 | package/import/Git/NAS/lake/runtime 仍未完成。 |
| DQ-CP8-CR060-02 | follow_up_tracking | 后续 package/import/layout 是否进入 CR061？ | 保留 CR061 为下一迁移 CR。 | 推迟到 Git remote 之后。 | 推荐方案先解决代码身份一致性；备选先发布远端。 | 需要测试矩阵。 |
| DQ-CP8-CR060-03 | runtime_authorization | Git remote 初始化是否仍不由 CR060 授权？ | 不授权，交给 CR062。 | 在 CR060 继续执行 remote add/push。 | 推荐方案边界清晰；备选混入发布风险。 | 远端仍为空。 |
| DQ-CP8-CR060-04 | runtime_authorization | NAS / data lake / runtime 是否继续分 CR？ | 继续分 CR063-CR065。 | 合并到下一个 CR。 | 推荐方案回滚面小；合并 CR 风险大。 | 完整迁移仍未结束。 |
| DQ-CP8-CR060-05 | security | 是否继续禁止凭据读取和 CR046 恢复？ | 继续禁止。 | 恢复 CR046 或读取配置。 | 推荐方案符合暂停边界；备选需独立授权。 | 交易 runtime 不推进。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP8-CR060-001 | CR060 关闭不授权 Git remote add / push / tag / branch rename / history rewrite |
| NA-CP8-CR060-002 | 不授权 file move / rename / delete |
| NA-CP8-CR060-003 | 不授权 package/import rename |
| NA-CP8-CR060-004 | 不授权 NAS 数据 copy / move / delete / archive promote |
| NA-CP8-CR060-005 | 不授权 `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish、provider fetch |
| NA-CP8-CR060-006 | 不授权 `.env`、token、password、cookie、session、private key、账户或交易事实读取 |
| NA-CP8-CR060-007 | 不授权 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live |
| NA-CP8-CR060-008 | 不授权恢复或推进 CR046 CP7 |

请只回复以下三种之一：

approve

修改: <具体修改点>

reject
