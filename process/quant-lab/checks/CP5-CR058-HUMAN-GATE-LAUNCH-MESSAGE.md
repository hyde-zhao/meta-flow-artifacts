请审查：`process/checkpoints/CP5-CR058-RELAYOUT-GATE-BATCH-A-LLD-BATCH.md`

自动预检结论：PASS。S01/S05 technical-note 可审；S02/S03/S04 full-lld 已补齐并通过 CP5 自动预检。CP5 approve 只批准 CR058 Batch-A 设计证据，不授权真实 move / rewrite、NAS、data lake、QMT/MiniQMT runtime、git remote、凭据读取或恢复 CR046。

本轮待人工决策项：6

如果你回复 approve，表示你接受以下 6 项推荐方案，不表示授权任何真实 move / rewrite、NAS、data lake、QMT/MiniQMT runtime、git remote、凭据读取或恢复 CR046；后续即使进入 CP6，也只能做已批准的静态文档 / no-op 设计收敛，真实执行必须另行授权。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP5-CR058-01 | implementation | CP5 是否只批准设计证据，不批准真实迁移执行？ | 是。 | 允许 CP6 实迁；不推荐。 | 防止误执行。 |
| DQ-CP5-CR058-02 | implementation | 是否接受 S02/S03/S04 full-lld 设计证据？ | 接受。 | 要求继续修改 LLD。 | 接受后仍不得真实执行。 |
| DQ-CP5-CR058-03 | architecture | 是否接受 candidate list schema？ | 接受 allowlist-first。 | repo-wide rewrite。 | 决定 candidate manifest。 |
| DQ-CP5-CR058-04 | risk_acceptance | 是否接受 preserve-audit allowlist？ | 接受。 | 单项移出。 | 保护历史审计。 |
| DQ-CP5-CR058-05 | implementation | rollback_ref 是否为执行硬前置？ | 接受 commit + bundle + manifest。 | commit-only。 | 缺 ref fail-closed。 |
| DQ-CP5-CR058-06 | runtime_authorization | CP5 是否不授权外部 / runtime / git remote / 凭据 / CR046 恢复？ | 确认不授权。 | 单项授权需独立 CR。 | 保持安全边界。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP5-CR058-01 | 真实 repo-local file move / rename / delete / bulk rewrite |
| NA-CP5-CR058-02 | 改写历史 `process/**`、checks、checkpoints、handoffs、历史 Story / LLD / DEV-LOG |
| NA-CP5-CR058-03 | git history rewrite、remote rename、git push、tag、force push |
| NA-CP5-CR058-04 | `git reset --hard`、`git checkout --` 回退用户未明确要求的工作树 |
| NA-CP5-CR058-05 | NAS mount / scan / mkdir / copy / delete / migration |
| NA-CP5-CR058-06 | `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish |
| NA-CP5-CR058-07 | provider fetch、真实数据拉取、untracked data bulk scan |
| NA-CP5-CR058-08 | `.env`、token、password、cookie、session、private key、账户凭据读取 |
| NA-CP5-CR058-09 | QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live |
| NA-CP5-CR058-10 | 恢复 CR046 CP7 或推进 CR046 runtime verification |
| NA-CP5-CR058-11 | 从 CR053 继承任何运行授权 |

approve

修改: <具体修改点>

reject
