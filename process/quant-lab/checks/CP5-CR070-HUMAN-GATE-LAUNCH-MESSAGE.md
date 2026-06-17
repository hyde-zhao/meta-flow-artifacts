请审查：process/checkpoints/CP5-CR070-GIT-GOVERNANCE-READINESS.md

自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP5-CR070-GIT-GOVERNANCE-CONTEXT.yaml
本轮待人工决策项：5

决策收集覆盖：已扫描 6 个来源，发现候选问题 11 个，纳入待决策 5 个；N/A / 缺失来源 2 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR070-01 | implementation | CP5 readiness 是否批准为本地准备通过、仍不执行？ | 批准 readiness，但不授权远端执行。 | 保持 pending；直接执行远端治理。 | 推荐方案符合边界。 | 后续仍需单独授权。 |
| DQ-CP5-CR070-02 | runtime_authorization | CP5 approve 是否授权任何 Git remote write？ | 不授权；所有 write 动作必须另有明确运行授权。 | 授权 push/tag；授权 branch/default branch。 | 推荐方案最安全。 | 防止误操作远端仓库。 |
| DQ-CP5-CR070-03 | security | history rewrite / force push / remote deletion / set-url 是否继续禁止？ | 继续禁止。 | 允许 force push；允许 deletion；允许 set-url。 | 推荐方案保守可回退。 | 保护远端历史和配置。 |
| DQ-CP5-CR070-04 | follow_up_tracking | CR070 active 状态如何记录？ | active formal CR / CP2-CP3-CP5 review pending；CR046 paused 保持。 | 关闭 CR070；恢复 candidate。 | 推荐方案反映启动事实。 | 影响 STATE / CR-INDEX。 |
| DQ-CP5-CR070-05 | risk_acceptance | 是否接受 CP5 后仍保留 default branch residual risk？ | 接受，直到后续 explicit execution authorization。 | 立即治理；取消治理。 | 推荐方案遵守本轮限制。 | 远端事实不变。 |

不授权项：

- 不授权 git remote add/set-url/push/tag。
- 不授权 branch rename、default branch governance、history rewrite、force push 或 remote deletion。
- 不授权 NAS/data lake/provider/catalog/runtime 操作。
- 不授权读取 `.env`、token、账号、密码、cookie、session、private key、SSH agent 细节或交易事实。
- 不授权 physical move、bulk import rewrite 或 CR046 recovery。

approve
修改: <具体修改点>
reject
