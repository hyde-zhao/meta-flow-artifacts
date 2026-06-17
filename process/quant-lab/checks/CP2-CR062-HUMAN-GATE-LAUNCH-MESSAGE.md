请审查：process/checkpoints/CP2-CR062-REQUIREMENTS-BASELINE.md
自动预检结论：PASS
上下文胶囊：process/context/CP2-CR062-REQUIREMENT-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 17 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR062-01 | scope | 是否正式启动 CR062？ | 启动 CR062，CR046 继续暂停。 | 保持 candidate；先 bulk relayout；先 NAS/lake。 | 推荐先建立 GitHub 可追踪基线。 | 只启动门禁，不执行 Git 写动作。 |
| DQ-CP2-CR062-02 | scope | 范围是否限定为 repository publication gate？ | 限定 GitHub publication，不含 physical/NAS/lake/runtime。 | 合并 relayout；合并 NAS/lake；合并 runtime。 | 推荐风险分层。 | 后续 CR 继续。 |
| DQ-CP2-CR062-03 | security | 是否接受 include/exclude policy？ | 接受，post-approval 冻结 manifest。 | 全仓库发布；或 code-only。 | 推荐兼顾审计和安全。 | 需要扫描。 |
| DQ-CP2-CR062-04 | runtime_authorization | CP5 前是否禁止 Git 写动作？ | 禁止 remote/add/commit/push/tag/rename/rewrite。 | 立即 remote；立即 commit/push。 | 推荐遵守门禁。 | 发布延后一轮。 |
| DQ-CP2-CR062-05 | follow_up_tracking | 后续迁移顺序如何保留？ | Git publication 后再分 CR 做 relayout/NAS/lake/runtime。 | 先 relayout；先 NAS/lake；恢复 CR046。 | 推荐保持可追踪。 | 完整迁移未完成。 |

不授权项：
- CP2 approve 不授权 Git remote add / set-url、commit、push、tag、branch rename、history rewrite 或 force push。
- 不授权 NAS/data lake/provider/catalog 操作。
- 不授权凭据读取、QMT/MiniQMT runtime 或恢复 CR046。
- 不授权 physical move / rename / delete 或 bulk import rewrite。

回复：
approve
修改: <具体修改点>
reject
