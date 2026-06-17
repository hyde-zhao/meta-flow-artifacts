请审查：process/checkpoints/CP2-CR068-REPOSITORY-HYGIENE-BASELINE.md

自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP2-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml
本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR068-01 | scope | 是否正式启动 CR068？ | 启动 CR068。 | 保持候选；继续接受风险；恢复 CR046。 | 推荐方案关闭残余风险。 | 新增门禁但边界清晰。 |
| DQ-CP2-CR068-02 | scope | 范围是否限定为 `.gitignore` forward-fix？ | 仅处理 tracked `.gitignore` public baseline forward-fix。 | 合并 branch/NAS/lake/runtime。 | 推荐方案最小。 | 后续治理需独立 CR。 |
| DQ-CP2-CR068-03 | security | 是否接受 `.gitignore` 需扫描？ | 接受，先扫描再 commit/push。 | 不发布或不扫描。 | 推荐方案安全。 | scan fail 则停止。 |
| DQ-CP2-CR068-04 | follow_up_tracking | CR062 如何处理？ | 保持 blocked，CR068 只做 forward-fix。 | 回 CR062；标记 superseded。 | 推荐方案不重开 blocker。 | CR062 历史保留。 |
| DQ-CP2-CR068-05 | risk_acceptance | CP2 是否不授权执行？ | 确认只批准范围。 | CP2 直接授权执行。 | 推荐方案遵守门禁。 | 执行延后到 CP5 后。 |

不授权项：

- 不授权修改 `.gitignore` 本体。
- 不授权 git clone/fetch、git add、git commit、git push、git tag、remote deletion、remote set-url。
- 不授权 branch/default branch governance、force push、history rewrite。
- 不授权 `.env` / credential 读取、NAS/data lake/provider/catalog/runtime、physical move、bulk import rewrite 或 CR046 recovery。

approve
修改: <具体修改点>
reject
