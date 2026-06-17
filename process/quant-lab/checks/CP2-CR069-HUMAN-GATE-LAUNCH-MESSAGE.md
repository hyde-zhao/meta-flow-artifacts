请审查：process/checkpoints/CP2-CR069-SUPERSESSION-BASELINE.md

自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP2-CR069-SUPERSESSION-CONTEXT.yaml
本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR069-01 | scope | 是否正式启动 CR069？ | 启动 CR069。 | 保持候选；直接关闭 CR062；恢复 CR046。 | 推荐方案先建门禁。 | 不改 CR062。 |
| DQ-CP2-CR069-02 | follow_up_tracking | CR062 替代关系如何表达？ | 表达为被 CR066/067/068 替代。 | 保持 blocked wording；标记 failed。 | 推荐方案解释完整。 | 影响 CR tracking。 |
| DQ-CP2-CR069-03 | scope | 是否排除 branch governance？ | 排除，另起 CR。 | 合并治理；忽略风险。 | 推荐方案守边界。 | master residual risk 保留。 |
| DQ-CP2-CR069-04 | security | CP2 是否不授权外部操作？ | 确认只批准范围。 | 直接授权执行；授权 Git。 | 推荐方案合规。 | 无外部状态变化。 |
| DQ-CP2-CR069-05 | risk_acceptance | CP5 前 CR062 是否仍 blocked？ | 接受。 | 立即改状态；放弃收口。 | 推荐方案避免未审先改。 | 短期 still blocked。 |

不授权项：

- 不授权修改 CR062 正式状态或 CR tracking 分类。
- 不授权 git remote add/set-url/push/tag、branch/default branch governance、history rewrite 或 force push。
- 不授权读取 `.env`、token、账号、密码、cookie、session、private key、SSH agent 细节或交易事实。
- 不授权 NAS/data lake/provider/catalog/runtime、physical move、bulk import rewrite 或 CR046 recovery。

approve
修改: <具体修改点>
reject
