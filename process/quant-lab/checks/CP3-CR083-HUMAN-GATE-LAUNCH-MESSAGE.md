请审查：process/checkpoints/CP3-CR083-REMOTE-PUSH-HLD-REVIEW.md
自动预检结论：PASS
上下文胶囊：process/context/CP3-CR083-REMOTE-PUSH-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：7
决策收集覆盖：已扫描 STATE、CP2 checkpoint、CP3 自动预检、CR083 formal CR 和本地 Git facts；候选问题 31 项，纳入待决策 7 项；discussion log N/A。
如果你回复 approve，表示你接受以下 7 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR083-01 | architecture | 远端发布架构？ | two-step fail-closed：preflight first, push later。 | 直接 push；no-op；clean route。 | 推荐可验证。 | 发布延后一次确认。 |
| DQ-CP3-CR083-02 | security | remote URL 如何处理？ | CP5 approve 后读取，输出脱敏。 | 不读取；完整打印。 | 推荐兼顾验证和隐私。 | URL 可能敏感。 |
| DQ-CP3-CR083-03 | runtime_authorization | 是否允许网络远端查询？ | 不允许 fetch/ls-remote，只读本地 metadata。 | 允许 ls-remote；fetch。 | 推荐避免外部影响。 | 不能证明远端实时状态。 |
| DQ-CP3-CR083-04 | implementation | push target 如何限定？ | 只考虑当前分支 existing upstream；无 upstream 停止。 | 自动设置 upstream；push master/main。 | 推荐不写 config。 | 无 upstream 时阻断。 |
| DQ-CP3-CR083-05 | security | high-risk Git 操作？ | 禁止 default/tag/force/history/set-url/delete。 | 允许 tag；force/rewrite；set-url。 | 推荐保护远端历史。 | default branch 未治理。 |
| DQ-CP3-CR083-06 | risk_acceptance | dirty worktree 是否阻断？ | 不阻断 preflight，输出说明。 | 先清理；停止。 | 推荐推进 baseline。 | 可能误解 push 范围。 |
| DQ-CP3-CR083-07 | follow_up_tracking | preflight 失败如何处理？ | fail-closed，写证据并返回人工决策。 | 自动修复；force push；set-url。 | 推荐安全。 | 可能需要后续 CR。 |
不授权项：
- 不授权 git push。
- 不授权 git fetch、git ls-remote 或网络远端查询。
- 不授权自动设置 upstream、remote add/set-url/delete。
- 不授权 default branch governance、branch rename、tag、force push 或 history rewrite。
- 不授权 amend、squash、merge、rebase、reset。
- 不授权 external process Git/NAS、data/reports、凭据、NAS 内容、provider/lake/catalog、runtime、CR046、cleanup。
approve
修改: <具体修改点>
reject
