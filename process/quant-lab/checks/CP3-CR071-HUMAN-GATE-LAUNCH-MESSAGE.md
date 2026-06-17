请审查：process/checkpoints/CP3-CR071-HLD-REVIEW.md

自动预检结论：PASS
上下文胶囊：process/context/CP3-CR071-DESIGN-CONTEXT.yaml
本轮待人工决策项：5

决策收集覆盖：已扫描 6 个来源，发现候选问题 14 个，纳入待决策 5 个；N/A / 缺失来源 2 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR071-01 | architecture | copy-first 是否作为主架构？ | copy-first shadow root，不 rename/move old root。 | rename/move；snapshot-only。 | 推荐方案可回退。 | 影响目标目录可用性。 |
| DQ-CP3-CR071-02 | implementation | 是否复制 `.git`？ | 复制 `.git`，target 可作为本地 Git 工作树。 | 不复制 `.git`；后续 init/clone。 | 推荐方案保留本地状态。 | 复制本地 Git config 但不远端写。 |
| DQ-CP3-CR071-03 | implementation | target root 存在时如何处理？ | 非空则阻断，不覆盖。 | 覆盖合并；删除；改名。 | 推荐方案保护已有内容。 | 避免误覆盖。 |
| DQ-CP3-CR071-04 | risk_acceptance | partial target 如何回退？ | 停止并记录，不自动删除。 | 自动删除；自动重试。 | 推荐方案避免二次破坏。 | 可能需后续人工清理。 |
| DQ-CP3-CR071-05 | architecture | 是否允许路径引用或环境切换？ | 不允许，本轮只复制。 | 同步切 shell/IDE/env；批量改写路径。 | 推荐方案影响面小。 | root cutover 后置。 |

不授权项：

- 不授权 rename/move/delete old root。
- 不授权覆盖、删除或清理已存在 target root。
- 不授权 remote Git write、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 NAS/data lake/provider/catalog/runtime/credential、`.env` read/copy、QMT / MiniQMT runtime 或 CR046 recovery。
- 不授权 shell/IDE/cron/systemd/环境变量切到新 root。

approve
修改: <具体修改点>
reject
