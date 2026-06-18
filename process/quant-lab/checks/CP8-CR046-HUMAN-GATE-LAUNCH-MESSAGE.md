# CP8-CR046 Human Gate Launch Message

请审查：`process/checkpoints/CP8-CR046-DELIVERY-READINESS.md`

自动预检结论：`PASS_WITH_RISK`。CR046 的 CP6 implementation 为 `PASS`，CP7 静态 / fixture / 文档 / 契约验证为 `PASS_WITH_RISK`。本次 CP8 只确认 framework-first 文档 / 契约交付和剩余风险接受，不授权任何运行时或外部环境动作。

Context Capsule：`process/context/CP8-CR046-DELIVERY-CONTEXT.yaml`
Release Context：`process/release/RELEASE-CONTEXT-CR046.yaml`

决策收集覆盖：已扫描 `STATE.md` human gate 队列、CR046 CP7 verification report、test report、review/fixes、release context 和当前用户“继续”指令；纳入 5 个待人工决策项。`STATE.md` 中无既有 CR046 CP8 未决项，本次 checkpoint 新增 DQ-CP8-CR046-01..05。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权下面列出的禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 推荐方案 |
|---|---|---|
| DQ-CP8-CR046-01 | risk_acceptance | 接受 `READY_WITH_RISK`，关闭 CR046 framework-first 当前交付。 |
| DQ-CP8-CR046-02 | scope | 关闭范围仅限文档 / 契约 / fixture / 静态验证，不包含具体策略、runner 或 runtime。 |
| DQ-CP8-CR046-03 | runtime_authorization | CP8 不授权 QMT/MiniQMT/XtQuant/gateway、NAS、凭据、账户、submit/cancel、simulation/live。 |
| DQ-CP8-CR046-04 | implementation | runner 未开发不阻断 CR046；runner 研究 / 设计 / 实现交给后续 CR091 或等价独立 CR。 |
| DQ-CP8-CR046-05 | follow_up_tracking | 保留 CR047/CR048/CR049/CR050 候选，CR089/CR091 保持 blocked，不自动启动。 |

不授权项：

- 不启动 QMT/MiniQMT/XtQuant/gateway，不连接、不安装、不运行。
- 不访问 NAS。
- 不读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- 不执行 submit/cancel、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不自动启动 CR089 或 CR091。

请回复以下三者之一：

approve
修改: <具体修改点>
reject
