请审查：process/checkpoints/CP3-CR062-HLD-REVIEW.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP3-CR062-DESIGN-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 32 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR062-01 | architecture | 是否 gate-first fail-closed？ | approve -> preflight/scan -> manifest -> commit -> remote -> push。 | 门禁前 remote precheck；跳过 scan。 | 推荐审计完整。 | scan 失败 BLOCKED。 |
| DQ-CP3-CR062-02 | security | scan gate 是否 fail-closed？ | BLOCKING/HIGH 即停止。 | 只警告；只 secret scan。 | 推荐更安全。 | 可能阻断发布。 |
| DQ-CP3-CR062-03 | implementation | branch 是否保持当前分支？ | 不 rename，冲突时停止。 | rename main；新建发布分支。 | 推荐最少侵入。 | 可能需再决策。 |
| DQ-CP3-CR062-04 | implementation | process/docs evidence 是否纳入？ | 选择性纳入 CR058-CR062 evidence。 | code-only；全量 process/docs。 | 推荐可追踪且控风险。 | 需要 manifest 审查。 |
| DQ-CP3-CR062-05 | security | remote handling 是否 fail-safe？ | absent 时 add exact target；conflict 时停止。 | set-url；secondary remote。 | 推荐避免误改远端。 | remote 冲突 BLOCKED。 |

不授权项：
- CP3 approve 不授权 Git remote add / set-url、commit、push、tag、branch rename、history rewrite 或 force push。
- 不授权 NAS/data lake/provider/catalog 操作。
- 不授权凭据读取、QMT/MiniQMT runtime 或恢复 CR046。
- 不授权 physical move / rename / delete 或 bulk import rewrite。

回复：
approve
修改: <具体修改点>
reject
