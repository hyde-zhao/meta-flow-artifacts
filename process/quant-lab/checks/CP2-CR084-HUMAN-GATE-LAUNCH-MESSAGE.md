请审查：process/checkpoints/CP2-CR084-REMOTE-TARGET-BASELINE.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP2-CR084-REMOTE-TARGET-CONTEXT.yaml（read_profile=compact，status=ready）
上下文胶囊：process/context/CP2-CR084-REMOTE-TARGET-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：7
决策收集覆盖：已扫描 STATE、CR083 handoff、CP2 自动预检、本地 Git facts 和当前用户输入；候选问题 17 项，纳入待决策 7 项；discussion log N/A。
如果你回复 approve，表示你接受以下 7 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR084-01 | scope | 是否启动 CR084？ | 启动 remote target / push gate，不执行远端动作。 | 暂停；恢复 CR046；直接 push。 | 推荐可审计；直接 push 风险高。 | 远端仍不变。 |
| DQ-CP2-CR084-02 | implementation | baseline 是否限定 `d4d2881`？ | 是，只围绕 `d4d2881`。 | 先提交 dirty worktree；改用其他 commit。 | 推荐边界清晰。 | 未提交文件不会发布。 |
| DQ-CP2-CR084-03 | security | remote URL 是否必须用户确认？ | 是，不从历史或记忆推断。 | 复用历史 URL；自动猜测。 | 推荐防误推。 | 需要用户提供 URL。 |
| DQ-CP2-CR084-04 | runtime_authorization | CP2 是否授权远端动作？ | 不授权，后置 CP5。 | CP2 授权全部；只授权 ls-remote。 | 推荐先决策后动作。 | CP2 不改变外部状态。 |
| DQ-CP2-CR084-05 | security | default/tag/force/history 是否纳入？ | 全部排除。 | 纳入默认分支；tag；force/rewrite。 | 推荐保护远端历史。 | 默认分支治理未完成。 |
| DQ-CP2-CR084-06 | follow_up_tracking | external process Git/NAS 是否处理？ | 不处理，保持本地 ledger。 | 本轮治理 process；NAS-only；Git+NAS。 | 推荐不混合治理线。 | process 远端备份仍未决。 |
| DQ-CP2-CR084-07 | risk_acceptance | dirty worktree 是否阻断？ | 不阻断门禁，但必须提醒。 | 先清理；停止。 | 推荐推进 target gate。 | 可能误解 push 范围。 |
不授权项：
- 不授权 git ls-remote、git fetch 或网络远端查询。
- 不授权 remote add/set-url/delete、set-upstream、git push 或 git push -u。
- 不授权从历史、记忆或配置隐式推断 remote URL。
- 不授权 default branch、branch rename、tag、force push 或 history rewrite。
- 不授权 dirty worktree / 未跟踪历史迁移文档提交。
- 不授权 external process Git/NAS、data/reports、凭据、NAS 内容、provider/lake/catalog、runtime、CR046、old-root cleanup。
approve
修改: <具体修改点>
reject
