请审查：process/checkpoints/CP5-CR083-REMOTE-PUSH-READINESS.md
自动预检结论：PASS
上下文胶囊：process/context/CP5-CR083-REMOTE-PUSH-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：8
决策收集覆盖：已扫描 STATE、CP2/CP3 checkpoint、CP5 自动预检、本地 Git facts 和 CR083 formal CR；候选问题 41 项，纳入待决策 8 项；discussion log N/A。
如果你回复 approve，表示你接受以下 8 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR083-01 | runtime_authorization | 是否允许 read-only preflight？ | 允许一次 status/branch/branch-vv/remote-v/log-1。 | 不读取 remote；网络查询；同时 push。 | 推荐取得事实且不写远端。 | 会读取 remote URL。 |
| DQ-CP5-CR083-02 | security | remote URL 如何展示？ | 用户可见输出脱敏。 | 完整打印；完全不展示。 | 推荐兼顾审计与隐私。 | 仍可能暴露仓库名。 |
| DQ-CP5-CR083-03 | runtime_authorization | 是否允许网络远端查询？ | 不允许 fetch/ls-remote。 | 允许 ls-remote；fetch。 | 推荐不触发认证/网络风险。 | 不能证明远端最新状态。 |
| DQ-CP5-CR083-04 | runtime_authorization | 是否允许 git push？ | 本次不授权；preflight 后二次确认。 | 同时 push existing upstream；push 指定目标。 | 推荐避免误推。 | 远端仍不变。 |
| DQ-CP5-CR083-05 | implementation | baseline 是否只允许 `d4d2881`？ | 是，只证明 `d4d2881`。 | 纳入 dirty worktree；amend/squash/rebase。 | 推荐可追溯。 | dirty files 不发布。 |
| DQ-CP5-CR083-06 | security | 是否允许修改 remote/upstream/default branch？ | 不允许。 | 自动设置 upstream；set-url；default branch cutover。 | 推荐不改配置。 | 无 upstream 会阻断。 |
| DQ-CP5-CR083-07 | security | tag/force/history 是否允许？ | 全部不允许。 | tag；force push；rewrite。 | 推荐保护历史。 | 不创建 tag。 |
| DQ-CP5-CR083-08 | follow_up_tracking | preflight 后如何推进？ | 输出脱敏 summary 并停止等待 push/no-push 决策。 | 自动 push；自动修复。 | 推荐符合 gate 边界。 | 需要下一次确认。 |
不授权项：
- 不授权 git push。
- 不授权 git fetch、git ls-remote 或网络远端查询。
- 不授权 remote add/set-url/delete、set-upstream、default branch governance、branch rename。
- 不授权 tag、force push、history rewrite、amend、squash、merge、rebase、reset。
- 不授权提交 README.md、docs/USER-MANUAL.md、pyproject.toml、uv.lock 或历史未跟踪迁移文档。
- 不授权 external process Git/NAS、data/reports、凭据、NAS 内容、provider/lake/catalog、runtime、CR046、old-root cleanup。
approve
修改: <具体修改点>
reject
