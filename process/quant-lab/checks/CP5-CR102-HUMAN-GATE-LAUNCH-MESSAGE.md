# CP5 CR102 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-READINESS.md`

自动预检结论：PASS。阻断项：0 for CP5 design-only / 1 for real execution。

Context Capsule Summary：`process/context/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-CONTEXT.yaml`（read_profile=compact）。

决策收集覆盖：已扫描 10 个来源，发现候选问题 48 个，纳入待决策 5 个；N/A / 缺失来源 1 类，原因见 checkpoint 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权任何真实 NAS/env/runtime/trading/publish 动作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR102-01 | implementation | 是否批准当前 CP5 design-only 设计包？ | 批准 DESIGN / TEST-PLAN / TASKS / full LLD，后续只允许本地账本和设计收敛。 | 修改后重提；reject 回 CP3 approved；只保留 HLD。 | 推荐方案让后续授权门禁可计算；备选会延后或缺少细节。 | 影响 CP5 状态、future gate 输入和审计链。 |
| DQ-CP5-CR102-02 | security | path/export/root/mount/identity 与 permission 是否继续 user-explicit-only？ | 保持全部 `UNSET_BY_USER` / not-authorized；agent 不发现、不探测、不读取 env。 | 用户提供值但仍不访问；暂停线下确认；授权 agent 探测。 | 推荐方案安全边界最强；agent 探测不推荐。 | 防止路径泄漏、凭据读取和 NAS 探测。 |
| DQ-CP5-CR102-03 | runtime_authorization | CP5 approve 是否授权真实 NAS/env/runtime/trading/publish 动作？ | 不授权；CP5 approve 只批准设计包，真实动作全部继续 false。 | 另起 per-run NAS check gate；用户手工回传脱敏 evidence；授权 agent 代跑。 | 推荐方案符合当前约束；后两者需要独立授权。 | 防止 CP5 被误读为运行许可。 |
| DQ-CP5-CR102-04 | risk_acceptance | 是否接受 CP5 后 CR102 仍 blocked-for-execution？ | 接受 PASS_WITH_RISK：设计包 ready，但真实 NAS path/mount/permission/operation 仍未授权。 | 阻塞到真实授权；关闭为 blocked；取消真实 NAS gate。 | 推荐方案保留可审计进展；备选会停止或关闭路线。 | 不能声称真实 NAS ready。 |
| DQ-CP5-CR102-05 | follow_up_tracking | future check/publish/pull/copy/delete 等如何分流？ | 全部保持独立 future runtime gate，不自动启动；publish/delete/write/copy 需 high-risk gate。 | 合并 check；合并 publish/pull/copy；全部取消。 | 推荐方案边界最窄；合并写类操作风险高。 | 影响 RA-CR101-003 后续台账和 CR089 overlap。 |

不授权项：

- 不访问、列取、读取、复制、写入、发布、拉取、删除、检查或挂载真实 NAS。
- 不读取凭据/env/账户、资金、持仓、委托、成交或原始日志。
- 不启动 QMT/MiniQMT/XtQuant/gateway runtime。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不恢复 CR089 或 CR020，不自动启动 QMT direct-run、MiniQMT gateway 或 order-write gate。

可回复：

approve

修改: <具体修改点>

reject
