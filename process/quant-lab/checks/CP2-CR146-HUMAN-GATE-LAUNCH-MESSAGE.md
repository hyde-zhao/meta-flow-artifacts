请审查：process/checkpoints/CP2-CR146-HIGH-RISK-LAKE-ACTIONS.md
自动预检结论：PASS
上下文胶囊：process/context/CP2-CR146-HIGH-RISK-LAKE-ACTIONS-CONTEXT.yaml，状态 ready，read_profile=compact。
Context Capsule Summary：见 checkpoint 的 `### Context Capsule Summary`，默认 capsule-first，全文档读取扩展 0 次。

审批者摘要：
- 本次确认服务的整体目标：确认 CR-139 post-close 真实 lake/runtime 验证完成后，是否接受高风险动作继续后置，并允许 CR-146 当前 validation delivery 以 READY_WITH_RISK 收口。
- 推荐动作：approve 默认方案，即不执行 N1 物理分区迁移、不执行 legacy delete/archive、不对 business-conflict groups 做语义择优。
- approve 后会发生什么：CR-146 当前 validation delivery 可关闭为 READY_WITH_RISK；高风险候选留在 follow-up tracking。
- approve 不授权什么：不授权 provider/NAS/runtime/trading/credential/Git remote、物理迁移、legacy 删除/archive movement、business-conflict 语义选择或 catalog/pointer republish。
- 不确认会阻塞什么：阻塞 CR-146 关闭和 CR-139 post-close 授权验证收口；不阻塞当前 data-lake current truth 读取使用。

本轮待人工决策项：4
决策收集覆盖：已扫描 8 个来源，发现候选问题 46 个，纳入待决策 4 个；N/A / 缺失来源 2 类，原因见 checkpoint 的 Decision Collection Coverage。
Decision Collection Coverage：见 checkpoint 的 `### Decision Collection Coverage`。
决策分层：必须用户决策 2 项；高风险策略确认 1 项；agent 默认处理 1 项；仅审计记录 1 项。
如果你回复 approve，表示你接受以下 4 项推荐方案，不表示授权以下禁止操作。

待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR146-01 | risk_acceptance | 是否执行 N1 physical partition migration？ | 不执行，保留 current catalog/published pointer/fail-closed guard。 | A. 只做新数据新分区；B. 全量 copy-first migration。 | 推荐风险最低；备选治理更强但需要 rollback/quiescent window。 | 高，影响全 lake physical layout。 |
| DQ-CP2-CR146-02 | risk_acceptance | 是否对 business-conflict groups 做语义选择并写入？ | 不做语义择优，保持 full-group quarantine/fail-closed。 | A. 逐 dataset full-group quarantine write；B. 制定 semantic rule 后择优。 | 推荐避免污染历史事实；备选需要业务规则和抽样审查。 | 高，4,272,624 business-conflict groups。 |
| DQ-CP2-CR146-03 | runtime_authorization | 是否授权 provider catalog、NAS sync、legacy archive/delete？ | 不授权。 | A. NAS dry-run；B. retain+superseded metadata；C. delete/archive。 | 推荐权限最小；备选涉及外部系统或不可逆删除。 | 高，涉及外部写入和删除。 |
| DQ-CP2-CR146-04 | follow_up_tracking | 是否允许 CR146 当前 validation delivery 关闭？ | 允许关闭为 READY_WITH_RISK，高风险项留台账。 | A. 保持 active；B. 拆多个正式 CR 立即推进。 | 推荐状态最清晰；备选扩大范围或制造假阻塞。 | 中，影响后续调度。 |

不授权项：
- provider fetch
- NAS sync
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/live/trading
- credential read
- Git remote write
- physical partition migration
- legacy delete/archive movement
- business-conflict semantic selection
- catalog/pointer republish

approve
修改: <具体修改点>
reject
