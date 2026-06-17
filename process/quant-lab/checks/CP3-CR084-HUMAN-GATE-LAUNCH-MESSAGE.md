请审查：process/checkpoints/CP3-CR084-REMOTE-TARGET-HLD-REVIEW.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP3-CR084-REMOTE-TARGET-CONTEXT.yaml（read_profile=compact，status=ready）
上下文胶囊：process/context/CP3-CR084-REMOTE-TARGET-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：7
决策收集覆盖：已扫描 STATE、CR084 formal CR、CP3 自动预检、CR083 preflight；候选问题 25 项，纳入待决策 7 项；discussion log N/A。
如果你回复 approve，表示你接受以下 7 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR084-01 | architecture | remote URL 来源如何定义？ | 用户提供或逐字确认 URL。 | 复用历史 URL；自动读取选择。 | 推荐防误推。 | URL 缺失时停止。 |
| DQ-CP3-CR084-02 | architecture | remote name 使用什么？ | 使用 `origin`。 | `github`；自定义名。 | 推荐符合常规。 | 影响命令和 upstream。 |
| DQ-CP3-CR084-03 | architecture | 目标分支如何选择？ | 推送当前分支到同名远端分支。 | `master`；`main`；自定义分支。 | 推荐保留当前分支语义。 | 可能不是默认分支。 |
| DQ-CP3-CR084-04 | security | 是否允许覆盖已有 remote？ | 不允许 set-url/remove，只考虑无 remote 时 add。 | 允许 set-url；删除重建。 | 推荐降低配置破坏风险。 | 发现已有 remote 即停止。 |
| DQ-CP3-CR084-05 | runtime_authorization | `ls-remote` 如何定位？ | 可选只读网络 precheck，CP5 单项授权。 | 必须执行；完全跳过。 | 推荐让用户选择网络风险。 | 可能触发认证。 |
| DQ-CP3-CR084-06 | security | push 策略如何限制？ | 只允许普通非 force push。 | force；tag；history rewrite。 | 推荐保护远端历史。 | 可能无法完成发布。 |
| DQ-CP3-CR084-07 | follow_up_tracking | external process ledger 是否纳入？ | 不纳入，独立治理。 | 建立 process remote；Git+NAS。 | 推荐缩小风险。 | process ledger 仍本地化。 |
不授权项：
- 不授权 CP3 阶段执行 git ls-remote、remote add、set-upstream 或 push。
- 不授权 remote set-url/delete、默认分支治理、branch rename。
- 不授权 tag、force push、history rewrite、amend、squash、merge、rebase、reset。
- 不授权 dirty worktree commit。
- 不授权 external process Git/NAS、data/reports、凭据、NAS 内容、provider/lake/catalog、runtime、CR046、old-root cleanup。
approve
修改: <具体修改点>
reject
