请审查：process/checkpoints/CP3-CR066-CLEAN-PUBLICATION-HLD-REVIEW.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP3-CR066-CLEAN-PUBLICATION-DESIGN-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 28 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR066-01 | architecture | clean publication 架构？ | clean snapshot / allowlist manifest。 | current branch；orphan branch；history rewrite。 | 推荐风险可控。 | publication 延迟。 |
| DQ-CP3-CR066-02 | security | `.env.example` 是否 conditional allow？ | placeholder-only / no-secret scan PASS 后允许。 | 始终排除；始终允许。 | 推荐平衡。 | 需要扫描。 |
| DQ-CP3-CR066-03 | implementation | reports/runs 是否默认排除？ | 默认排除。 | 风险接受后发布；私有仓库；历史清理。 | 推荐符合源码发布面。 | 过程证据不公开。 |
| DQ-CP3-CR066-04 | security | NAS/internal topology 文件如何处理？ | 默认不纳入，必要时 redaction。 | 原样纳入；删除本地；私有文档。 | 推荐减少暴露。 | 需 redaction。 |
| DQ-CP3-CR066-05 | implementation | 是否允许 Git 写动作？ | 不允许，只输出策略。 | 创建 clean branch；commit docs；remote add。 | 推荐最小授权。 | 不推进远端。 |

不授权项：
- 不授权 `git add`、`git commit`、`git remote add`、`git push`。
- 不授权 `git tag`、branch creation / rename、history rewrite、force push、remote deletion。
- 不授权 NAS/data lake/provider/catalog/runtime/凭据操作。
- 不授权 physical move / bulk import rewrite 或 CR046 recovery。

回复：
approve
修改: <具体修改点>
reject
