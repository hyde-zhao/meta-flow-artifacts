请审查：process/checkpoints/CP3-CR070-GIT-GOVERNANCE-HLD-REVIEW.md

自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP3-CR070-GIT-GOVERNANCE-CONTEXT.yaml
本轮待人工决策项：5

决策收集覆盖：已扫描 6 个来源，发现候选问题 11 个，纳入待决策 5 个；N/A / 缺失来源 2 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR070-01 | architecture | 是否采用 staged authorization 治理架构？ | 采用计划优先、分阶段授权、fail-closed。 | 只记录风险；直接执行远端治理。 | 推荐方案可审计。 | 影响 CP5 / 执行前置条件。 |
| DQ-CP3-CR070-02 | architecture | branch/default branch 目标策略如何冻结？ | 记录为目标决策，但当前远端分支不变。 | 立即切到 main；永久接受 master。 | 推荐方案先决策后授权。 | residual risk 保留。 |
| DQ-CP3-CR070-03 | security | remote write 控制策略如何设计？ | 所有 write 动作必须有后续明确 runtime_authorization 和 preflight。 | CP3 approve 即授权 push；只口头确认。 | 推荐方案可验证。 | 防止误推送。 |
| DQ-CP3-CR070-04 | security | history rewrite / force push / remote deletion / set-url 策略？ | 默认禁止。 | 允许 force push；允许 deletion；允许 set-url。 | 推荐方案风险最低。 | 保护远端历史与配置。 |
| DQ-CP3-CR070-05 | risk_acceptance | 是否接受 CP3 通过不改变远端事实？ | 接受；CP3 只确认治理设计。 | 要求立即执行；取消治理。 | 推荐方案符合本轮限制。 | 远端仓库保持现状。 |

不授权项：

- 不授权执行 CP5 readiness 或任何远端 Git 写动作。
- 不授权 git remote add/set-url/push/tag。
- 不授权 branch rename、default branch governance、history rewrite、force push 或 remote deletion。
- 不授权 NAS/data lake/provider/catalog/runtime、credential read、physical move、bulk import rewrite 或 CR046 recovery。

approve
修改: <具体修改点>
reject
