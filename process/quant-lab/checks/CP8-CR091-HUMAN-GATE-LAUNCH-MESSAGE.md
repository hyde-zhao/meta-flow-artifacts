# CP8-CR091 Human Gate Launch Message

请审查：`process/checkpoints/CP8-CR091-DELIVERY-READINESS.md`

自动预检结论：`PASS_WITH_RISK`。CR091 的 CP6 implementation 为 `PASS`，CP7 离线 contract / fixture / checker / static verification 为 `PASS_WITH_RISK`。本次 CP8 只确认 CR091 离线 runner 合同和实现切片的交付就绪与剩余风险接受，不授权任何运行时或外部环境动作。

Context Capsule：`process/context/CP8-CR091-DELIVERY-CONTEXT.yaml`
Release Context：`process/release/RELEASE-CONTEXT-CR091.yaml`

决策收集覆盖：已扫描 `STATE.md` human gate 队列、CR091 CP7 verification report、test report、review/fixes、release context、QA handoff 和当前用户“继续推进”指令；纳入 5 个待人工决策项。`STATE.md` 中无既有 CR091 CP8 未决项，本次 checkpoint 新增 DQ-CP8-CR091-01..05。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权下面列出的禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR091-01 | risk_acceptance | 是否接受 CR091 当前 CP8 结论？ | 接受 `READY_WITH_RISK`，关闭 CR091 当前离线 runner 交付。 | 补真实 runtime gate 后再关闭；reject 回退到 CP7 pending CP8。 | 推荐方案与 CP6/CP7 证据一致；补 runtime 会进入未授权范围。 | 接受后仍不声明真实 runtime ready。 |
| DQ-CP8-CR091-02 | scope | CR091 关闭范围是否限定为研究 / 设计 / 离线实现 / fixture 验证？ | 是，仅关闭离线 runner 合同和实现切片。 | 扩大到真实 QMT runner；扩大到 NAS package exchange；扩大到 submit/cancel。 | 推荐方案边界清楚；扩大范围需要新设计和授权。 | 真实交易主机和策略可交易性仍未证明。 |
| DQ-CP8-CR091-03 | runtime_authorization | CP8 approve 是否授权 QMT/MiniQMT/XtQuant/gateway/runner、NAS、凭据、账户或交易动作？ | 不授权。 | 授权只读 runtime smoke；授权 NAS package exchange；授权 submit/cancel。 | 推荐方案符合当前安全边界；备选风险高且需要独立门禁。 | 后续真实验证仍 blocked。 |
| DQ-CP8-CR091-04 | follow_up_tracking | 后续候选如何分流？ | 保留 runtime readonly smoke、NAS package exchange、order-write、ledger hygiene 为候选，不自动启动。 | 本轮强制启动某个候选；取消全部候选。 | 推荐方案保留路线且不扩大执行面；立即启动会越过当前 CP8 授权。 | 后续仍需用户明确选择和新门禁。 |
| DQ-CP8-CR091-05 | implementation | 是否接受当前无需额外用户手工 runtime 验证即可关闭离线切片？ | 接受；CP8 只关闭离线验证通过的 runner 合同。 | 要求用户在交易主机手工验证后再关；取消 runner 交付。 | 推荐方案不要求用户执行未授权真实环境；手工验证会引入 runtime / 凭据 / 账户边界。 | 无真实 runtime 证据风险进入接受项。 |

不授权项：

- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway / runner runtime。
- 不访问、列取、读取、复制、拉取、写入、发布或删除 NAS。
- 不读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- 不执行 submit / cancel、buy / sell。
- 不执行 simulation / live。
- 不执行 provider fetch、lake write、catalog publish。
- 不自动启动 CR089。
- 不恢复 CR020 用户删除的 gateway 路线。

请回复以下三者之一：

approve
修改: <具体修改点>
reject
