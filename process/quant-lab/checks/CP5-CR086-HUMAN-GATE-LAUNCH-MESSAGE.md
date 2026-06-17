请审查：process/checkpoints/CP5-CR086-LEDGER-CONVERGENCE-READINESS.md
自动预检结论：PASS
Context Capsule 摘要：process/context/CP5-CR086-LEDGER-CONVERGENCE-CONTEXT.yaml（read_profile=compact，status=ready）
本轮待人工决策项：10
决策收集覆盖：已扫描 STATE pending queue、CR086 formal CR、CP2 checkpoint、CP3 checkpoint、CP5 auto check、CR084/CR073/CR074 formal CR 和当前用户输入；候选问题 14 项，纳入待决策 10 项；discussion log N/A，因为 CR086 是 ledger-only 状态收敛。
如果你回复 approve，表示你接受以下 10 项推荐方案，不表示授权以下不授权项。

待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR086-01 | scope | CR086 是否只处理台账状态收敛？ | 仅处理 CR084/CR073/CR074、STATE、CR-INDEX。 | 同时处理远端归档；停止 CR086。 | 推荐聚焦且符合 process 软连接边界。 | 不解决远端归档。 |
| DQ-CP2-CR086-02 | security | 是否继续不授权远端、.env、data/reports/NAS/runtime？ | 继续不授权。 | 一并授权远端或数据操作。 | 推荐最小风险。 | 只能做 ledger-only。 |
| DQ-CP2-CR086-03 | scope | 顶层 active_change 是否保持 CR046？ | 保持 CR046 user-paused。 | 切到 CR086；恢复 CR046。 | 推荐避免误恢复 CR046。 | CR086 只记录在 cr_tracking。 |
| DQ-CP3-CR086-01 | architecture | CR084 最终语义如何收敛？ | 标记为 closed-superseded-by-cr085。 | 保持 blocked；关闭为 cancelled。 | 推荐准确表达被 CR085 替代。 | CR084 不再等待执行。 |
| DQ-CP3-CR086-02 | architecture | CR073 achieved 状态如何收敛？ | 关闭为 closed-current-delivery / READY_WITH_RISK。 | 保持 active achieved；仅移出查询视图。 | 推荐与 readiness 完成事实对齐。 | 后续 root/data/runtime 独立。 |
| DQ-CP3-CR086-03 | architecture | CR074 achieved 状态如何收敛？ | 关闭为 closed-current-delivery / READY_WITH_RISK。 | 保持 active achieved；仅移出查询视图。 | 推荐与 logical root authority 完成事实对齐。 | 旧根退役和远端归档仍未授权。 |
| DQ-CP5-CR086-01 | implementation | approve 后是否允许执行 ledger-only 更新？ | 允许更新 STATE、CR-INDEX、CR084、CR073、CR074。 | 只生成计划；停止执行。 | 推荐可完成收敛。 | 仅修改 process ledger。 |
| DQ-CP5-CR086-02 | risk_acceptance | 是否接受 CR073/CR074 关闭为 READY_WITH_RISK？ | 接受，风险项转入后续独立 CR。 | 要求补 CP8；保持 active achieved。 | 推荐降低误导。 | 风险需后续追踪。 |
| DQ-CP5-CR086-03 | follow_up_tracking | 后续 candidates 如何处理？ | 保留候选 / spike，不启动。 | 同时启动某个候选；取消全部候选。 | 推荐不扩大范围。 | 后续 backlog 仍存在。 |
| DQ-CP5-CR086-04 | runtime_authorization | CP5 是否授权任何外部动作？ | 不授权，只允许 ledger-only 文件更新和校验命令。 | 授权远端 / 数据 / runtime。 | 推荐符合当前目标。 | 无外部副作用。 |

不授权项：
- 不授权任何新的 git push、remote deletion、force push、history rewrite、tag、default branch governance、remote add/set-url/remove。
- 不授权读取、打印、保存、复制或上传 .env、token、password、private key、cookie、session。
- 不授权 current dirty worktree commit。
- 不授权 data/reports 内容读取、NAS/provider/lake/catalog/runtime、CR046 recovery、old root retirement 或 cleanup。

approve
修改: <具体修改点>
reject
