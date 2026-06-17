请审查：process/checkpoints/CP8-CR061-BATCH-B-DELIVERY-READINESS.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/release/RELEASE-CONTEXT-CR061.yaml（read_profile=compact）
决策收集覆盖：已扫描 4 个来源，发现候选问题 15 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，并接受 CR061 Batch B READY_WITH_RISK；不表示授权以下 8 项禁止操作。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR061-01 | risk_acceptance | 是否接受 READY_WITH_RISK 并关闭本批交付？ | 接受并关闭 CR061 Batch B。 | 补 full pytest；或 reject 回滚。 | 推荐完成第一切片；补测更稳但更慢。 | 后续 CR 仍需继续。 |
| DQ-CP8-CR061-02 | risk_acceptance | 是否接受 full pytest 未运行？ | 接受；作为后续风险。 | 现在 full pytest；或扩大离线 subset。 | 推荐符合授权边界；补测覆盖更强。 | 非 import 回归未完全覆盖。 |
| DQ-CP8-CR061-03 | follow_up_tracking | 后续迁移 CR 顺序如何处理？ | 下一步 CR062 Git remote cutover。 | 先 bulk relayout；或先 NAS/lake。 | 推荐先建立远端基线。 | 远端仍未初始化。 |
| DQ-CP8-CR061-04 | security | CP8 approve 是否仍不授权外部/运行/凭据/CR046？ | 确认不授权。 | 合并 Git；或合并 NAS/lake/runtime。 | 推荐边界清晰；备选风险大。 | 完整迁移继续分 CR。 |
| DQ-CP8-CR061-05 | implementation | 是否保留 legacy roots 并推迟 physical relayout？ | 保留，后续独立 CR。 | 现在 bulk move；或永久不做。 | 推荐兼顾兼容和迁移。 | 源码目录仍非最终形态。 |

不授权项：
- NA-CP8-CR061-001 不执行真实 release / push / publish。
- NA-CP8-CR061-002 不执行 Git remote add / push / tag / branch rename / history rewrite。
- NA-CP8-CR061-003 不执行 NAS 数据 copy / move / delete / archive promote。
- NA-CP8-CR061-004 不执行 data lake root replacement、provider fetch、lake write、catalog publish。
- NA-CP8-CR061-005 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。
- NA-CP8-CR061-006 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。
- NA-CP8-CR061-007 不恢复或推进 CR046 CP7。
- NA-CP8-CR061-008 不执行 physical move / rename / delete 或 bulk import rewrite。

回复：
approve
修改: <具体修改点>
reject
