请审查：process/checkpoints/CP2-CR078-GIT-PUBLICATION-BASELINE.md
自动预检结论：PASS
上下文胶囊：process/context/CP2-CR078-GIT-PUBLICATION-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：7
决策收集覆盖：已扫描 STATE、CR082 handoff、CP2 自动预检、CR078 formal CR、CR-INDEX 和当前用户显式选择题；候选问题 24 项，纳入待决策 7 项；discussion log N/A。
如果你回复 approve，表示你接受以下 7 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR078-01 | scope | 是否启动 CR078？ | 启动，只做 scope gate。 | 保持 candidate；取消。 | 推荐承接 CR082。 | staged removals 有治理入口。 |
| DQ-CP2-CR078-02 | scope | staged process removals 是否入范围？ | 纳入，CP5 再决定提交。 | 恢复；立即提交。 | 推荐不越权。 | 1572 deletions 保持 staged。 |
| DQ-CP2-CR078-03 | implementation | pointer files 是否同范围？ | 纳入同一 scope。 | 单独提交；排除。 | 推荐保持恢复闭环。 | 影响新 clone 恢复。 |
| DQ-CP2-CR078-04 | implementation | CR081/CR082 docs 如何处理？ | docs/quality 和 docs/release 候选纳入；process/release 保留外部 ledger。 | 全纳入；全排除。 | 推荐遵守外部化。 | 影响审计证据。 |
| DQ-CP2-CR078-05 | runtime_authorization | CP2 是否授权 commit/push？ | 不授权。 | CP2 授权 commit；授权 push。 | 推荐分层。 | 避免误提交/推送。 |
| DQ-CP2-CR078-06 | security | remote/default/tag/force/history 是否默认执行？ | 默认不授权。 | 纳入治理；允许 tag/force/rewrite。 | 推荐保护远端。 | 不可逆风险。 |
| DQ-CP2-CR078-07 | follow_up_tracking | 外部 process 如何治理？ | 另起门禁决定 Git/NAS/hybrid。 | 本轮建 Git；本轮推 NAS。 | 推荐保持边界。 | 外部 ledger 远端化延后。 |
不授权项：
- 不授权 git add / git commit / git push。
- 不授权 remote URL / 默认分支 / branch rename / tag / force push / history rewrite。
- 不授权 data/reports/.env/凭据/NAS/provider/lake/catalog/runtime/CR046/old-root cleanup。
approve
修改: <具体修改点>
reject
