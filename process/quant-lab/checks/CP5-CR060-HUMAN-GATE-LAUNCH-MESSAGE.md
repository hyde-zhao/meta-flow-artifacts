请审查：`process/checkpoints/CP5-CR060-EXECUTION-READINESS.md`

自动预检结论：PASS_WITH_RISK。CR060 execution manifest 已冻结，当前只等待人工授权；未授权前不会执行真实文本改写。

Context Capsule 摘要：CR059 已完成 NAS mount、systemd unit、目录骨架、cold backup 和 Git bundle；CR058 已完成 static-only/no-op gate；CR046 保持用户暂停，不恢复。

决策收集覆盖摘要：已扫描 CR060 formal CR、execution manifest、move plan、preserve-audit decisions、rollback ref、CR058 candidate/preserve-audit/rollback 输入和 CR059 manifest seed。候选问题数 6，纳入待决策数 6，N/A 0。

本轮待人工决策项：6

如果你回复 approve，表示你接受以下 6 项推荐方案，并授权 CR060-ACT-001 / CR060-ACT-002 两项文档文本改写；不表示授权以下 10 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR060-01 | runtime_authorization | 是否授权执行 CR060-ACT-001 / ACT-002？ | 授权执行 README 和 USER-MANUAL 文档身份改写。 | 只保留门禁不执行。 | 推荐方案开始真实迁移；备选继续停留在计划。 | 只影响两个用户文档。 |
| DQ-CP5-CR060-02 | scope | 执行范围是否禁止扩展到其他文件？ | 禁止扩展；只改 manifest 两项。 | 允许临场追加文件。 | 推荐方案可审计；备选易越界。 | 未覆盖文件留给后续 CR。 |
| DQ-CP5-CR060-03 | risk_acceptance | 是否接受复用 CR059 rollback refs？ | 接受 docs-only 复用；HEAD 变化则刷新。 | 执行前创建新 bundle。 | 推荐方案足够且快；备选更强但增加操作。 | 若 HEAD 变化必须阻断。 |
| DQ-CP5-CR060-04 | security | 是否确认不读取凭据和 `.env`？ | 确认不读取。 | 读取配置辅助判断。 | 推荐方案符合安全边界；备选不必要且高风险。 | 无法验证真实运行配置。 |
| DQ-CP5-CR060-05 | implementation | 是否保留 `MARKET_DATA_LAKE_ROOT` 和 package/import 不变？ | 保留不变。 | 同步替换为 `QUANT_LAB_*` 或改包名。 | 推荐方案避免破坏运行；备选需要 CR061/CR064。 | 部分旧标识会保留。 |
| DQ-CP5-CR060-06 | runtime_authorization | 是否确认本次不执行 Git remote / push / NAS / runtime？ | 确认不执行。 | 合并执行远端、NAS 或 runtime。 | 推荐方案边界清晰；备选风险高且需更多前置。 | 远端和数据迁移延后。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP5-CR060-001 | manifest 两项之外的任何 text rewrite |
| NA-CP5-CR060-002 | file move / rename / delete |
| NA-CP5-CR060-003 | 历史 `process/**`、checks、checkpoints、handoffs、Story、LLD、DEV-LOG 或历史 CR 改写 |
| NA-CP5-CR060-004 | `pyproject.toml`、import path、package name、dependency lock 改动 |
| NA-CP5-CR060-005 | `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish、provider fetch |
| NA-CP5-CR060-006 | NAS 数据 copy / move / delete / archive promote |
| NA-CP5-CR060-007 | Git remote add / push / tag / branch rename / history rewrite |
| NA-CP5-CR060-008 | `.env`、token、password、cookie、session、private key、账户或交易事实读取 |
| NA-CP5-CR060-009 | QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live |
| NA-CP5-CR060-010 | 恢复或推进 CR046 CP7 |

请只回复以下三种之一：

approve

修改: <具体修改点>

reject
