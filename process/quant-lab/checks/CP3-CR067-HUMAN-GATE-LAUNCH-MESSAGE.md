# CP3 CR067 Human Gate Launch Message

请审查：process/checkpoints/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-HLD-REVIEW.md

自动预检结论：PASS_WITH_RISK

Context Capsule：process/context/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml

Decision Collection Coverage：已扫描 pyproject、CR061 package/import 候选清单、CR067 execution plan、manifest、scan gate 与 CP3 capsule；候选问题 8 项，纳入待人工决策 5 项，tag/default branch rename/history rewrite 均作为不授权项处理。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权 Git 写动作、远端操作、history rewrite、force push、tag、branch rename、remote deletion、NAS/data lake/runtime/凭据、physical move、bulk import rewrite 或 CR046 recovery。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP3-CR067-01 | architecture | 是否采用 temporary clean snapshot 架构？ | 采用临时 clean snapshot + exact remote policy。 | 当前分支发布；history rewrite；code-only。 | 需要临时目录和 manifest freeze。 |
| DQ-CP3-CR067-02 | implementation | 是否保留 legacy roots？ | 发布 `quant_lab/**` 及 legacy roots。 | 只发 `quant_lab/**`；bulk rewrite。 | 维持现有 import 契约。 |
| DQ-CP3-CR067-03 | security | scan gate 是否必须先于 Git 写动作？ | 必须，BLOCKING/HIGH fail-closed。 | Git commit 后再扫；warning only。 | 扫描失败则停止。 |
| DQ-CP3-CR067-04 | implementation | remote conflict policy 是否 fail-safe？ | exact remote，冲突即停止。 | set-url；force push；删除 remote。 | 可能因远端冲突 blocked。 |
| DQ-CP3-CR067-05 | risk_acceptance | 是否接受 no rewrite/force/tag/deletion？ | 接受，forward-fix only。 | 允许 rewrite/force/tag。 | 后续修正可能多一轮 CR。 |

不授权项：

- 不授权任何 Git 写动作，直到 CP2/CP3/CP5 全部批准。
- 不授权 history rewrite、force push、tag、branch creation / rename、remote deletion 或 remote set-url。
- 不授权 NAS/data lake/provider/catalog/runtime/凭据操作。
- 不授权 physical move、bulk import rewrite 或 CR046 recovery。

推荐回复：

approve

修改: <具体修改点>

reject
