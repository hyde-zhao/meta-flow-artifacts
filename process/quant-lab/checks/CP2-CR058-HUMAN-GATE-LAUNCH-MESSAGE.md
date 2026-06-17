请审查：`process/checkpoints/CP2-CR058-REQUIREMENTS-BASELINE.md`

自动预检结论：PASS。CR058 已从 `FU-CR053-01` 创建 formal CR；CR053 保持 closed-current-delivery；CR046 保持用户暂停的 CP6 PASS / ready-for-verification，不恢复。

本轮待人工决策项：6

如果你回复 approve，表示你接受以下 6 项推荐方案，不表示授权任何真实迁移、文件移动、路径替换、NAS、data lake、QMT/MiniQMT runtime、provider、git remote 或凭据读取操作。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR058-01 | scope | 是否从 `FU-CR053-01` 创建正式 CR058？ | 创建正式 CR058。 | 保持 candidate。 | 创建不代表执行授权。 |
| DQ-CP2-CR058-02 | scope | 范围是否仅限 relayout gate 设计？ | 仅设计，不执行。 | 扩展真实 move；不推荐。 | 防止越权。 |
| DQ-CP2-CR058-03 | architecture | 候选范围如何定义？ | allowlist-first。 | repo-wide rewrite。 | 保护历史审计。 |
| DQ-CP2-CR058-04 | risk_acceptance | 历史引用是否 preserve-audit？ | 默认 preserve-audit。 | 逐项重写。 | 保持审计链。 |
| DQ-CP2-CR058-05 | runtime_authorization | 是否不授权真实运行 / 迁移 / git remote？ | 确认不授权。 | 单项授权需独立 CR。 | 保持安全边界。 |
| DQ-CP2-CR058-06 | implementation | 是否恢复 CR046？ | 不恢复。 | 合并恢复；不推荐。 | 避免交易 runtime 误启动。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP2-CR058-01 | 真实文件 move / rename / delete / bulk rewrite |
| NA-CP2-CR058-02 | NAS mount / scan / mkdir / copy / delete / migration |
| NA-CP2-CR058-03 | `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish |
| NA-CP2-CR058-04 | git push / tag / remote rename / history rewrite |
| NA-CP2-CR058-05 | 凭据、`.env`、token、password、cookie、session、private key 读取 |
| NA-CP2-CR058-06 | QMT / MiniQMT runtime、账户查询或交易动作 |
| NA-CP2-CR058-07 | provider fetch、untracked data bulk scan |
| NA-CP2-CR058-08 | 恢复或推进 CR046 CP7 |

approve

修改: <具体修改点>

reject

