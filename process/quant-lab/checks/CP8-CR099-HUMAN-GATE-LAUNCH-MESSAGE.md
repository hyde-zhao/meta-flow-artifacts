请审查：`process/checkpoints/CP8-CR099-DELIVERY-READINESS.md`

自动预检结论：`READY_WITH_RISK`，阻断项 0。

Context Capsule：`process/context/CP8-CR099-DELIVERY-CONTEXT.yaml`，状态 ready-for-human-review，read_profile=compact。

决策收集覆盖：已扫描 6 个来源，发现候选问题 17 个，纳入待决策 5 个；N/A / 缺失来源 0 个，详情见 checkpoint 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR099-01 | risk_acceptance | 是否接受 CR099 release_decision？ | 接受 `READY_WITH_RISK`，关闭当前 runner real readonly smoke 交付。 | `READY`；`NOT_READY`；保持 active 等待复测。 | 推荐方案诚实记录空持仓 / 交易日覆盖缺口；备选会弱化风险或延后收口。 | 当前交付可关闭，风险转 follow-up。 |
| DQ-CP8-CR099-02 | risk_acceptance | 是否接受 CP8 剩余风险？ | 接受空持仓路径、交易日路径未证明。 | 本轮补非空；交易日复测；取消当前交付。 | 推荐方案不扩大授权；补测需要新的账户状态 / 时间窗口和运行授权。 | 后续不能声称非空 / 交易日已证明。 |
| DQ-CP8-CR099-03 | runtime_authorization | CP8 approve 是否授权额外真实运行或外部访问？ | 不授权；只确认当前 evidence 和风险接受。 | 授权非空复测；授权 NAS；授权 order-write。 | 推荐方案权限最小；备选均需独立高风险门禁。 | 防止 CP8 被误读为额外运行授权。 |
| DQ-CP8-CR099-04 | follow_up_tracking | 后续事项如何分流？ | 后续候选保留，不自动启动。 | 立即启动非空复测；立即启动 NAS；立即启动 order-write。 | 推荐方案关闭当前 CR 且保留追踪；立即启动会扩大当前范围。 | 后续候选仍需用户明确选择。 |
| DQ-CP8-CR099-05 | scope | 是否关闭 CR099 当前交付？ | CP8 approve 后关闭为 `closed-current-delivery / READY_WITH_RISK`。 | 保持 active 等待更多证据；取消 CR099。 | 推荐方案清晰切分当前 smoke 与后续复测；保持 active 易混淆授权边界。 | 关闭后新增真实运行必须另起授权。 |

不授权项：

- 新的 QMT / MiniQMT / XtQuant / gateway / runner runtime run。
- Windows `.env`、凭据、真实账号、账户原文、证券代码、数量、资金、持仓明细、委托、成交或原始日志。
- NAS access / list / read / copy / pull / write / publish / delete。
- submit/cancel、buy/sell、simulation/live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089、NAS gate、order-write gate 或非空 / 交易日复测。

请直接回复以下任一整行：

approve

修改: <具体修改点>

reject
