请审查：process/checkpoints/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP5-CR061-LLD-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 38 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 10 项禁止操作。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR061-01 | runtime_authorization | 是否批准当前 CP5 门禁包？ | 批准门禁包，不立即执行真实改动。 | 补 scan 后再审；或 reject。 | 推荐进入实现准备；补 scan 更保守。 | 后续仍需实现授权。 |
| DQ-CP5-CR061-02 | implementation | 第一实现切片是否 metadata + compatibility namespace？ | 限定 Batch B，不 bulk move。 | 直接 bulk move；或只改 metadata。 | 推荐风险/价值平衡。 | legacy roots 保留。 |
| DQ-CP5-CR061-03 | risk_acceptance | CR059 bundle 是否只作 planning rollback？ | 接受；实现前刷新或 waiver。 | 现在新 bundle；或无 bundle 直接做。 | 推荐不扩大当前动作；无 bundle 不可接受。 | 实现前 BLOCKED。 |
| DQ-CP5-CR061-04 | security | preserve-audit / sensitive filter 是否继续有效？ | 有效，不改历史证据，不读敏感正文。 | 单项豁免；或批量历史改写。 | 推荐保护审计；批量改写风险高。 | 历史旧名保留。 |
| DQ-CP5-CR061-05 | runtime_authorization | CP5 approve 是否仍不授权 Git/NAS/lake/provider/runtime/凭据/CR046？ | 确认不授权。 | 合并 Git；或合并 NAS/lake/runtime。 | 推荐风险分层；备选回滚面大。 | 后续 CR 继续。 |

不授权项：
- NA-CP5-CR061-001 当前不修改 `pyproject.toml`、`uv.lock` 或源码目录。
- NA-CP5-CR061-002 不执行 file move / rename / delete。
- NA-CP5-CR061-003 不执行 bulk import rewrite。
- NA-CP5-CR061-004 不改写历史 `process/**`、checks、checkpoints、handoffs、Story、LLD、DEV-LOG 或历史 CR。
- NA-CP5-CR061-005 不执行 `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish、provider fetch。
- NA-CP5-CR061-006 不执行 NAS 数据 copy / move / delete / archive promote。
- NA-CP5-CR061-007 不执行 Git remote add / push / tag / branch rename / history rewrite。
- NA-CP5-CR061-008 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。
- NA-CP5-CR061-009 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。
- NA-CP5-CR061-010 不恢复或推进 CR046 CP7。

回复：
approve
修改: <具体修改点>
reject
