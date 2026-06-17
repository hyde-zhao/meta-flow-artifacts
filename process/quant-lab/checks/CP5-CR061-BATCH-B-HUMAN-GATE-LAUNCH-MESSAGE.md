请审查：process/checkpoints/CP5-CR061-BATCH-B-IMPLEMENTATION-AUTHORIZATION.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP5-CR061-BATCH-B-AUTHORIZATION-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 6 个来源，发现候选问题 39 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，并授权 CR061 Batch B 窄实现；不表示授权以下 10 项禁止操作。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR061B-01 | runtime_authorization | 是否授权 Batch B 窄实现？ | 仅授权 metadata、lock、可选 `quant_lab` namespace 和离线 smoke test。 | 只 metadata + lock；或继续 gate-only。 | 推荐能推进 package identity；备选更保守但推进少。 | 会产生 repo-local diff。 |
| DQ-CP5-CR061B-02 | implementation | dirty worktree include/exclude 是否作为执行边界？ | 采用 Batch B include/exclude manifest。 | 先 clean worktree；或扩大 include。 | 推荐可小步推进；扩大 include 风险高。 | 误纳入历史证据会阻断。 |
| DQ-CP5-CR061B-03 | risk_acceptance | 是否接受轻量 diff rollback，不刷新 Git bundle？ | 接受；记录 pre/post diff 和 smoke evidence。 | 先刷新 bundle；或无证据直接做。 | 推荐不触发 Git/NAS；无证据不可接受。 | diff 超范围则停止。 |
| DQ-CP5-CR061B-04 | security | 是否继续禁止 Git/NAS/lake/provider/runtime/凭据/CR046？ | 确认继续禁止。 | 合并 Git；或合并 NAS/lake/runtime。 | 推荐风险分层；备选回滚面大。 | 完整迁移仍需后续 CR。 |
| DQ-CP5-CR061B-05 | risk_acceptance | 验证是否限定为离线 import smoke 和必要 pytest subset？ | 接受离线 subset；不触发外部面。 | 强制 full pytest；或只做静态检查。 | 推荐覆盖关键风险；只静态不足。 | 可能留下非 import 回归风险。 |

不授权项：
- NA-CP5-CR061B-001 不执行 physical move / rename / delete。
- NA-CP5-CR061B-002 不执行 bulk import rewrite。
- NA-CP5-CR061B-003 不删除 legacy roots。
- NA-CP5-CR061B-004 不改写历史 `process/**`、checks、checkpoints、handoffs、Story、LLD、DEV-LOG 或历史 CR。
- NA-CP5-CR061B-005 不执行 Git remote add / push / tag / branch rename / history rewrite。
- NA-CP5-CR061B-006 不执行 NAS 数据 copy / move / delete / archive promote。
- NA-CP5-CR061B-007 不执行 `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish、provider fetch。
- NA-CP5-CR061B-008 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。
- NA-CP5-CR061B-009 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。
- NA-CP5-CR061B-010 不恢复或推进 CR046 CP7。

回复：
approve
修改: <具体修改点>
reject
