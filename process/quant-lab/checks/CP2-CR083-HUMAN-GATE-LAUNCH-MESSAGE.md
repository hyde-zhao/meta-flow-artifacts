请审查：process/checkpoints/CP2-CR083-REMOTE-PUSH-BASELINE.md
自动预检结论：PASS
上下文胶囊：process/context/CP2-CR083-REMOTE-PUSH-CONTEXT.yaml（read_profile=compact）
本轮待人工决策项：7
决策收集覆盖：已扫描 STATE、CR078 checkpoint、CP2 自动预检、本地 Git facts 和当前用户输入；候选问题 15 项，纳入待决策 7 项；discussion log N/A。
如果你回复 approve，表示你接受以下 7 项推荐方案，不表示授权以下不授权项。
待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR083-01 | scope | 是否启动 CR083？ | 启动 remote push preflight gate，不默认 push。 | 暂停；直接 push；转默认分支治理。 | 推荐可审计；直接 push 风险高。 | 远端仍不变。 |
| DQ-CP2-CR083-02 | implementation | baseline 是否限定 `d4d2881`？ | 是，只围绕 `d4d2881`。 | 先提交 dirty worktree；改用其他 commit。 | 推荐边界清晰。 | 未提交文件不会发布。 |
| DQ-CP2-CR083-03 | security | CP2 是否读取 remote/upstream？ | 不读取，后置到 CP5 approve。 | CP2 即读取；网络查询。 | 推荐先决策后动作。 | remote URL 可能敏感。 |
| DQ-CP2-CR083-04 | runtime_authorization | CP2 是否授权 push？ | 不授权。 | 直接 push；指定远端/分支 push。 | 推荐避免误推。 | 远端发布延后。 |
| DQ-CP2-CR083-05 | security | default/tag/force/history 是否纳入？ | 全部排除。 | 纳入默认分支；tag；force/rewrite。 | 推荐保护远端历史。 | 默认分支治理未完成。 |
| DQ-CP2-CR083-06 | follow_up_tracking | external process Git/NAS 是否处理？ | 不处理，保持本地 ledger。 | 本轮 git init；NAS-only；Git+NAS。 | 推荐不混合治理线。 | process 远端备份仍未决。 |
| DQ-CP2-CR083-07 | risk_acceptance | dirty worktree 是否阻断？ | 不阻断 preflight，但必须提醒。 | 先清理；停止。 | 推荐推进 baseline 验证。 | 可能误解 push 范围。 |
不授权项：
- 不授权 git push。
- 不授权 CP2 阶段读取 remote URL/upstream 或执行 git remote -v。
- 不授权 git fetch、git ls-remote 或网络远端查询。
- 不授权 remote add/set-url/delete、默认分支治理、branch rename、tag、force push 或 history rewrite。
- 不授权 dirty worktree / 未跟踪历史迁移文档提交。
- 不授权 external process Git/NAS、data/reports、凭据、NAS 内容、provider/lake/catalog、runtime、CR046、old-root cleanup。
approve
修改: <具体修改点>
reject
