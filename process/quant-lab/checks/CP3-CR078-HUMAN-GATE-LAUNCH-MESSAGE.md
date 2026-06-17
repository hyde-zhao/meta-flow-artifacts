请审查：process/checkpoints/CP3-CR078-GIT-PUBLICATION-HLD-REVIEW.md
自动预检结论：PASS
上下文胶囊：process/context/CP3-CR078-GIT-PUBLICATION-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：7
决策收集覆盖：已扫描 STATE、CP2 context、CP3 自动预检、CR078 formal CR、pointer files 和当前用户显式选择题；候选问题 25 项，纳入待决策 7 项；discussion log N/A。
如果你回复 approve，表示你接受以下 7 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR078-01 | architecture | Git/process 边界？ | Git 跟踪 pointer 和获批证据；process 正文外部化。 | process 重新入库；无 pointer。 | 推荐符合 CR082。 | 仓库边界清楚。 |
| DQ-CP3-CR078-02 | implementation | 提交形态？ | scoped commit 候选。 | 拆分；只提交 pointer。 | 推荐原子。 | 影响回滚粒度。 |
| DQ-CP3-CR078-03 | implementation | docs/release-context 分类？ | docs/quality 与 docs/release 候选；process/release 留 external ledger。 | force-add；不提交 docs。 | 推荐边界清楚。 | 审计可见性。 |
| DQ-CP3-CR078-04 | security | remote URL/default branch？ | 不执行，只记录禁区。 | 纳入治理。 | 推荐低风险。 | 远端配置安全。 |
| DQ-CP3-CR078-05 | security | tag/force/history？ | 禁止。 | 允许 tag；允许 force/rewrite。 | 推荐保护历史。 | 不可逆风险。 |
| DQ-CP3-CR078-06 | implementation | 当前分支策略？ | 不切换；如提交，仅当前分支本地提交。 | 切 master/main；新分支。 | 推荐不改变分支状态。 | 提交落点风险。 |
| DQ-CP3-CR078-07 | follow_up_tracking | 外部 process 治理？ | 另起门禁。 | 本轮 Git/NAS/hybrid。 | 推荐不扩大范围。 | 外部 ledger 仍本地。 |
不授权项：
- 不授权 git add / git commit / git push。
- 不授权 remote URL / 默认分支 / branch rename / tag / force push / history rewrite。
- 不授权外部 process Git init / NAS promote / Git+NAS 混合治理。
- 不授权 data/reports/.env/凭据/NAS/provider/lake/catalog/runtime/CR046/old-root cleanup。
approve
修改: <具体修改点>
reject
