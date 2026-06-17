# CP2 CR067 Human Gate Launch Message

请审查：process/checkpoints/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-BASELINE.md

自动预检结论：PASS_WITH_RISK

Context Capsule：process/context/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml

Decision Collection Coverage：已扫描 STATE、CR-INDEX、CR066 CP8、CR062 blocker、CR067 正式 CR 与 CP2 capsule；候选问题 7 项，纳入待人工决策 5 项，NAS/data lake/runtime/CR046 recovery 作为不授权项处理。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权 Git 写动作、远端操作、凭据读取、数据迁移、运行时连接、physical move、bulk import rewrite 或 CR046 recovery。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR067-01 | scope | 是否正式启动 CR067？ | 启动并登记为 active formal CR。 | 保持 candidate；回 CR062；恢复 CR046。 | 新增门禁但避免绕过 blocker。 |
| DQ-CP2-CR067-02 | scope | 范围是否限于 clean publication execution gate？ | 限定范围，不含 NAS/lake/runtime/CR046。 | 合并其他迁移面。 | 后续迁移需独立 CR。 |
| DQ-CP2-CR067-03 | security | 是否接受 allowlist/default-exclude policy？ | 接受，`.env.example` 条件纳入。 | 发布 current branch；code-only。 | 需要额外扫描。 |
| DQ-CP2-CR067-04 | risk_acceptance | 是否确认不发布 current branch as-is？ | 确认，使用临时 clean snapshot。 | 直接 push 当前分支。 | snapshot 构建失败则 blocked。 |
| DQ-CP2-CR067-05 | follow_up_tracking | 执行失败如何处理？ | fail-closed / forward-fix。 | 回 CR062 as-is；跳过扫描。 | 可能延迟 publication。 |

不授权项：

- 不授权 `git add/commit/remote add/remote set-url/push/tag`、branch creation / rename、history rewrite、force push 或 remote deletion。
- 不授权 NAS/data lake/provider/catalog/runtime/凭据操作。
- 不授权读取 `.env`、token、账号、密码、cookie、session、private key 或交易事实。
- 不授权 physical move、bulk import rewrite 或 CR046 recovery。

推荐回复：

approve

修改: <具体修改点>

reject
