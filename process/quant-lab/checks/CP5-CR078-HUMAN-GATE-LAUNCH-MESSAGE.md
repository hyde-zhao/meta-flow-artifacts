请审查：process/checkpoints/CP5-CR078-GIT-PUBLICATION-READINESS.md
自动预检结论：PASS
上下文胶囊：process/context/CP5-CR078-GIT-PUBLICATION-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：8
决策收集覆盖：已扫描 STATE、CP2/CP3 checkpoint、CP5 自动预检、Git status/branch、CR078 formal CR 和当前用户显式选择题；候选问题 32 项，纳入待决策 8 项；discussion log N/A。
如果你回复 approve，表示你接受以下 8 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR078-01 | implementation | staged process removals 是否提交？ | 允许纳入一次本地 scoped commit。 | 保持 staged；恢复；拆分。 | 推荐收敛 Git index。 | 改变本地提交内容。 |
| DQ-CP5-CR078-02 | implementation | pointer files 是否同提交？ | `.gitignore`、`LEDGER.md`、`ledger.yaml`、script 同提交。 | 单独提交；排除。 | 推荐恢复闭环。 | 影响 clone bootstrap。 |
| DQ-CP5-CR078-03 | implementation | CR081/CR082 docs 是否纳入？ | 纳入 docs/quality 和 docs/release 的 CR081/CR082；process/release 留外部 ledger。 | 全排除；force-add process/release；纳入所有历史 release docs。 | 推荐审计和边界兼顾。 | release-context 不进业务 Git。 |
| DQ-CP5-CR078-04 | runtime_authorization | 是否允许 git add allowlist + commit？ | 允许一次 allowlist git add 和一次本地 commit。 | 不允许；多 commit。 | 推荐给 staged removals 出口。 | 改变本地 history。 |
| DQ-CP5-CR078-05 | runtime_authorization | 是否允许 git push？ | 不授权 push，commit 后二次确认。 | 同时 push 当前分支；push 指定远端/分支。 | 推荐避免远端风险。 | 远端不变。 |
| DQ-CP5-CR078-06 | security | remote/default/tag/force/history？ | 全部不授权。 | 允许 default/tag/force/rewrite。 | 推荐保护远端。 | 远端治理延后。 |
| DQ-CP5-CR078-07 | follow_up_tracking | 外部 process 如何治理？ | 本轮只保留本地 external ledger；另起 Git/NAS/hybrid CR。 | 本轮 git init；NAS-only；Git+NAS。 | 推荐不扩大范围。 | 外部 ledger 远端化延后。 |
| DQ-CP5-CR078-08 | implementation | 当前分支是否提交落点？ | 若 commit，只在当前分支本地提交；不切分支。 | 切 master/main；新建分支；停止确认目标分支。 | 推荐不改变分支状态。 | 当前分支名有语义风险。 |
不授权项：
- 不授权 git push。
- 不授权 remote URL / remote set-url / 默认分支 / branch rename / tag / force push / history rewrite。
- 不授权切换分支、merge、rebase、squash、reset、restore staged removals。
- 不授权将 README.md、docs/USER-MANUAL.md、pyproject.toml、uv.lock 或历史未跟踪迁移文档纳入本轮提交，除非明确修改 allowlist。
- 不授权 force-add process/release/RELEASE-CONTEXT-CR081.yaml 或 CR082 到 quant-lab Git。
- 不授权 data/reports/.env/凭据/NAS/provider/lake/catalog/runtime/CR046/old-root cleanup。
approve
修改: <具体修改点>
reject
