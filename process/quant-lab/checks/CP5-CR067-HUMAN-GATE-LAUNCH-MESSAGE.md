# CP5 CR067 Human Gate Launch Message

请审查：process/checkpoints/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-READINESS.md

自动预检结论：PASS_WITH_RISK

Context Capsule：process/context/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml

Decision Collection Coverage：已扫描 CR067 正式 CR、execution plan、manifest、scan gate、CP2/CP3/CP5 自动检查、CP5 capsule 和 CR062 blocker evidence；候选问题 9 项，纳入待人工决策 5 项；凭据、token、SSH 细节、release tag、NAS/data lake/runtime 均不纳入授权。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案；并且只有在 CP2/CP3/CP5 均批准后，才授权受控 post-approval sequence：临时 clean snapshot、allowlist copy、fail-closed scan、临时 repo git init/add/commit、exact remote add 和 push。approve 不授权 force push、tag、history rewrite、remote deletion、NAS/data lake/runtime/凭据、physical move、bulk import rewrite 或 CR046 recovery。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP5-CR067-01 | runtime_authorization | 是否授权受控 post-approval sequence？ | CP2/CP3/CP5 全部 approve 后执行临时 snapshot、scan、临时 repo commit、exact remote add/push。 | 继续不执行；current branch push；force/tag/rewrite。 | 将产生受控 Git 写动作和外部 push。 |
| DQ-CP5-CR067-02 | implementation | 是否冻结 manifest？ | 冻结 allowlist/exclude/conditional include。 | 运行时临时增减；full process；code-only。 | 可能需后续 forward-fix。 |
| DQ-CP5-CR067-03 | security | 是否采用 fail-closed scans？ | no-secret/path/large/topology/cache 任一高风险即停止。 | 只扫 secret；warning only；push 后补扫。 | false positive 需修订后重审。 |
| DQ-CP5-CR067-04 | risk_acceptance | 验证范围是否 scoped？ | import smoke / py_compile / manifest-vs-snapshot / scan evidence / push evidence。 | 全量 pytest；不验证。 | 只证明 publication 基础可用。 |
| DQ-CP5-CR067-05 | follow_up_tracking | 失败如何处理？ | fail-closed，记录 blocker，forward-fix。 | force push；rewrite；remote deletion；回 CR062。 | 可能需要新增 CR。 |

不授权项：

- 不授权 current branch as-is publication。
- 不授权 force push、tag、history rewrite、branch creation / rename、remote deletion 或 remote set-url。
- 不授权读取或使用 `.env`、token、账号、密码、cookie、session、private key、SSH agent 细节或交易事实。
- 不授权 NAS/data lake/provider/catalog/runtime 操作。
- 不授权 physical move、bulk import rewrite 或 CR046 recovery。

推荐回复：

approve

修改: <具体修改点>

reject
