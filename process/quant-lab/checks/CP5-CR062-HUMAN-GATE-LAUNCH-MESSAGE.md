请审查：process/checkpoints/CP5-CR062-REPOSITORY-PUBLICATION-READINESS.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP5-CR062-REPOSITORY-PUBLICATION-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 35 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，并授权 CR062 在门禁批准后执行 DQ-CP5-CR062-01 列出的受控 Git publication；不表示授权以下 12 项禁止操作。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR062-01 | runtime_authorization | 是否批准 post-approval Git publication？ | 批准 preflight、scan、manifest、normal commit、remote add if absent/exact、push current branch。 | 只 preflight/scan；只 commit 不 push；reject。 | 推荐完成 GitHub baseline。 | approve 后会写 Git config / commit / GitHub remote。 |
| DQ-CP5-CR062-02 | implementation | commit manifest 是否 allowlist / denylist？ | 只 stage approved paths。 | `git add .`；code-only。 | 推荐控制风险；`git add .` 不可接受。 | manifest 需冻结。 |
| DQ-CP5-CR062-03 | security | scan 是否 commit/push 前硬门？ | BLOCKING/HIGH finding 阻断。 | 只记录继续；只 secret scan。 | 推荐更安全。 | 可能返工。 |
| DQ-CP5-CR062-04 | risk_acceptance | 是否接受 publication scoped regression？ | 重跑 CR061 smoke + `git diff --check`，full pytest 可选。 | 必须 full pytest；只 scan。 | 推荐不扩大外部面。 | 有残余回归风险。 |
| DQ-CP5-CR062-05 | risk_acceptance | 失败回滚是否 fail-closed / forward-fix？ | 不 rewrite、不 force、不删 remote，失败即停止。 | 允许 force push；自动 set-url/删除 remote。 | 推荐保护历史。 | 可能需要后续修复 CR。 |

不授权项：
- NA-CP5-CR062-001 CP5 approve 前不执行任何 Git 写动作。
- NA-CP5-CR062-002 不执行 `git tag`。
- NA-CP5-CR062-003 不执行 branch rename / default branch rename。
- NA-CP5-CR062-004 不执行 history rewrite / rebase-publication / force push。
- NA-CP5-CR062-005 不删除 remote 或远端 ref。
- NA-CP5-CR062-006 不使用 `git add .` 或提交 forbidden paths。
- NA-CP5-CR062-007 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。
- NA-CP5-CR062-008 不执行 NAS 数据 copy / move / delete / archive promote。
- NA-CP5-CR062-009 不执行 `MARKET_DATA_LAKE_ROOT` replacement、provider fetch、lake write、catalog publish。
- NA-CP5-CR062-010 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。
- NA-CP5-CR062-011 不恢复或推进 CR046 CP7。
- NA-CP5-CR062-012 不执行 physical move / rename / delete 或 bulk import rewrite。

回复：
approve
修改: <具体修改点>
reject
