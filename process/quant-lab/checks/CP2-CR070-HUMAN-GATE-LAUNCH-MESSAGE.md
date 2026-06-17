请审查：process/checkpoints/CP2-CR070-GIT-GOVERNANCE-BASELINE.md

自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP2-CR070-GIT-GOVERNANCE-CONTEXT.yaml
本轮待人工决策项：5

决策收集覆盖：已扫描 6 个来源，发现候选问题 11 个，纳入待决策 5 个；N/A / 缺失来源 2 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR070-01 | scope | 是否正式启动 CR070 gate？ | 启动为 active formal CR，并只生成本地门禁产物。 | 保持 candidate；取消候选。 | 推荐方案可审计且不执行。 | 影响 CR tracking。 |
| DQ-CP2-CR070-02 | follow_up_tracking | CR070 来源如何记录？ | 记录为 CR069 CP8 第 3 项决策后续 CR。 | 独立用户请求；并入 CR069。 | 推荐方案追溯清晰。 | 影响审计链。 |
| DQ-CP2-CR070-03 | security | 是否继续禁止远端 Git 写动作？ | 确认禁止 remote add/set-url/push/tag、branch/default branch、history rewrite、force push、remote deletion。 | 允许只读远端检查；允许部分治理。 | 推荐方案符合用户限制。 | 外部状态不变。 |
| DQ-CP2-CR070-04 | runtime_authorization | 是否继续禁止 NAS/lake/runtime/credential/CR046？ | 确认禁止，相关事项独立 CR。 | 合并 NAS/runtime；恢复 CR046。 | 推荐方案风险分层。 | 避免外部副作用。 |
| DQ-CP2-CR070-05 | risk_acceptance | 是否接受 default branch residual risk 暂不改变？ | 接受；本轮只冻结门禁。 | 立即治理；取消治理。 | 推荐方案可控。 | 远端仓库状态不变。 |

不授权项：

- 不授权 git remote add/set-url/push/tag。
- 不授权 branch rename、default branch governance、history rewrite、force push 或 remote deletion。
- 不授权 NAS/data lake/provider/catalog/runtime 操作。
- 不授权读取 `.env`、token、账号、密码、cookie、session、private key、SSH agent 细节或交易事实。
- 不授权 physical move、bulk import rewrite 或 CR046 recovery。

approve
修改: <具体修改点>
reject
