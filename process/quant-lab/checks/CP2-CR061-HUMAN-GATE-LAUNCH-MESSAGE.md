请审查：process/checkpoints/CP2-CR061-REQUIREMENTS-BASELINE.md
自动预检结论：PASS
上下文胶囊：process/context/CP2-CR061-REQUIREMENT-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 26 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR061-01 | scope | 是否正式启动 CR061？ | 启动 CR061，CR046 继续暂停。 | 保持 candidate；或先 CR062。 | 推荐按迁移顺序推进；备选会停滞或提前发布远端。 | 只启动门禁，不执行改动。 |
| DQ-CP2-CR061-02 | scope | 当前范围是否 gate-only？ | 只创建 CR、candidate、plan 和 CP gates。 | 立即实现；或只写 CR。 | 推荐可审计；立即实现风险高。 | 延后一轮实现。 |
| DQ-CP2-CR061-03 | risk_acceptance | dirty worktree 是否只用于 planning？ | 接受，真实实现前冻结 include/exclude。 | 先 clean/commit；或忽略 dirty state。 | 推荐不阻断门禁；忽略不可接受。 | 实现前仍 BLOCKED。 |
| DQ-CP2-CR061-04 | security | 是否继续禁止 Git/NAS/lake/runtime/凭据？ | 继续禁止。 | 合并 Git；或合并 NAS/lake/runtime。 | 推荐边界清晰；备选扩大风险。 | 后续 CR 增加。 |
| DQ-CP2-CR061-05 | implementation | 是否接受 staged compatibility-first？ | 先保留 legacy roots，再考虑 `quant_lab` bridge。 | bulk move；或永久 legacy roots。 | 推荐稳健；bulk 风险高；永久旧 roots 不收敛。 | 双入口共存。 |

不授权项：
- 不授权代码改动、file move/rename/delete、bulk import rewrite。
- 不授权 Git remote/push/tag/history rewrite。
- 不授权 NAS/data lake/provider/catalog 操作。
- 不授权凭据读取、QMT/MiniQMT runtime 或恢复 CR046。

回复：
approve
修改: <具体修改点>
reject
