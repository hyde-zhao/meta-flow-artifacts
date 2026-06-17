请审查：process/checkpoints/CP5-CR068-REPOSITORY-HYGIENE-READINESS.md

自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP5-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml
本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR068-01 | runtime_authorization | 是否授权受控 forward-fix sequence？ | 授权 exact checkout、scan、commit only `.gitignore`、fast-forward push。 | 不执行；合并 branch governance；force/rewrite。 | 推荐方案最小。 | 产生 Git 读写和外部 push。 |
| DQ-CP5-CR068-02 | implementation | 是否冻结 delta 为 `.gitignore`？ | 冻结为当前 tracked `.gitignore`。 | 临时增减；追加 docs。 | 推荐方案可审计。 | 其他 hygiene 文件暂缓。 |
| DQ-CP5-CR068-03 | security | 是否采用 scan fail-closed？ | 采用。 | warning；push 后补扫。 | 推荐方案安全。 | false positive 需修订。 |
| DQ-CP5-CR068-04 | risk_acceptance | 验证范围是否 scoped？ | 限定 delta/scan/remote/push evidence。 | 全量测试；不验证。 | 推荐方案匹配范围。 | 只证明 repo hygiene。 |
| DQ-CP5-CR068-05 | follow_up_tracking | 失败如何处理？ | fail-closed / forward-fix。 | force/rewrite/delete/回 CR062。 | 推荐方案低破坏。 | 可能需新增 CR。 |

不授权项：

- 不授权 branch/default branch governance。
- 不授权 force push、tag、history rewrite、remote deletion、remote set-url。
- 不授权 current branch as-is publication。
- 不授权 `.env` / credential 读取、NAS/data lake/provider/catalog/runtime、physical move、bulk import rewrite 或 CR046 recovery。

approve
修改: <具体修改点>
reject
