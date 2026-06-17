请审查：process/checkpoints/CP5-CR084-REMOTE-TARGET-READINESS.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP5-CR084-REMOTE-TARGET-CONTEXT.yaml（read_profile=compact，status=ready）
上下文胶囊：process/context/CP5-CR084-REMOTE-TARGET-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：10
决策收集覆盖：已扫描 STATE、CP2/CP3 checkpoint、CP5 自动预检、本地 Git facts 和 CR084 formal CR；候选问题 40 项，纳入待决策 10 项；discussion log N/A。
如果你回复 approve，表示你接受以下 10 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR084-01 | runtime_authorization | remote URL 如何提供？ | 用户提供或逐字确认 URL；未提供则不执行。 | 复用历史 URL；agent 推断。 | 推荐防误推。 | URL 缺失或疑似含凭据时停止。 |
| DQ-CP5-CR084-02 | implementation | remote name 是否使用 `origin`？ | 使用 `origin`。 | `github`；自定义名称。 | 推荐符合常规。 | 影响命令和 upstream。 |
| DQ-CP5-CR084-03 | implementation | target branch 如何选择？ | 推送当前分支到同名远端分支。 | `master`；`main`；自定义分支。 | 推荐保留当前分支语义。 | 同名分支可能不是默认分支。 |
| DQ-CP5-CR084-04 | runtime_authorization | 是否允许 `git ls-remote`？ | 允许一次只读 `git ls-remote`。 | 不执行网络检查；允许 fetch。 | 推荐验证目标且不写远端。 | 会访问网络并可能认证。 |
| DQ-CP5-CR084-05 | runtime_authorization | 是否允许 `git remote add`？ | 当前无 remote 且 URL 确认后允许 add。 | 只记录命令；允许 set-url。 | 推荐完成必要配置。 | 修改本地 Git 配置。 |
| DQ-CP5-CR084-06 | runtime_authorization | 是否允许 set-upstream？ | 仅随首次 push 使用 `git push -u`。 | 不设置 upstream；单独 set-upstream。 | 推荐一次动作完成。 | 修改本地 branch tracking。 |
| DQ-CP5-CR084-07 | runtime_authorization | 是否允许普通 push？ | 允许一次普通非 force push。 | 只 add remote；push 到 master/main。 | 推荐完成发布。 | 外部远端写入。 |
| DQ-CP5-CR084-08 | security | force/tag/history/default branch 是否允许？ | 全部不允许。 | tag；force；默认分支治理。 | 推荐保护远端历史。 | 不创建 release tag。 |
| DQ-CP5-CR084-09 | risk_acceptance | dirty worktree 是否继续排除？ | 继续排除，只发布 `d4d2881` 历史。 | 先提交；先清理。 | 推荐保持 baseline。 | 未提交文件不会发布。 |
| DQ-CP5-CR084-10 | follow_up_tracking | external process Git/NAS 与 CR046 如何处理？ | 全部不处理；CR046 保持 paused。 | 治理 process；恢复 CR046。 | 推荐避免混合风险。 | 后续治理仍未完成。 |
不授权项：
- 不授权使用未由你提供或逐字确认的 remote URL。
- 不授权读取 .env、token、API key、cookie、session、private key 或任何凭据。
- 不授权 git remote set-url、git remote remove、覆盖已有 remote。
- 不授权 git fetch、默认分支治理、branch rename。
- 不授权 tag、force push、history rewrite、amend、squash、merge、rebase、reset。
- 不授权 dirty worktree / 未跟踪历史迁移文档提交。
- 不授权 external process Git/NAS、data/reports、NAS 内容、provider/lake/catalog、QMT/MiniQMT runtime、CR046、old-root cleanup。
approve
修改: <具体修改点>
reject
