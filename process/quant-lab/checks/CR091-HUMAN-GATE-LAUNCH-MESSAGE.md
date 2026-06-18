# CR091 Human Gate Launch Message

请审查：

Checklist 路径：

- `process/checkpoints/CP2-CR091-QMT-STRATEGY-RUNNER-SCOPE-REVIEW.md`
- `process/checkpoints/CP3-CR091-QMT-STRATEGY-RUNNER-HLD-REVIEW.md`
- `process/checkpoints/CP5-CR091-QMT-STRATEGY-RUNNER-READINESS.md`

自动预检结论：

- CP2: `PASS_WITH_RISK`
- CP3: `PASS_WITH_RISK`
- CP5: `PASS_WITH_RISK`

Context Capsule：

- `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`
- `process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`
- `process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`

决策收集覆盖：

- 已扫描来源：STATE、CR091、CR089、CR046 CP8、CR091 research notes、CR091 HLD、CR091 LLD、CR091 Test Plan。
- 候选问题数：8。
- 纳入待决策数：8。
- N/A / 缺失原因：无。

本轮待人工决策项：8。

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR091-01 | scope | CR091 当前是否启动为静态研究 / 方案 / 实施计划门禁？ | 启动 CR091 门禁，但 CP5 前不实现、不运行。 | 暂停；直接 CR047；直接实现。 | 推荐方案解决 runner 结构缺口且保留权限边界；直接实现会绕过设计门禁。 | 不证明真实 runtime 可用。 | 用户要求先交付策略包时切 CR047。 |
| DQ-CP3-CR091-02 | architecture | runner 主体路线是什么？ | clean-room package-driven runner。 | 改造 lite-qmt-executor；接入 qmt-gateway/xqshare；接入 vn.py；仅手工脚本。 | 推荐方案依赖少、边界清晰；外部项目实盘接口面过大。 | 需要自行实现 package/cache/evidence。 | 未来复杂事件驱动需求出现时再评估 vn.py / Lean。 |
| DQ-CP3-CR091-03 | architecture | 多因子与其他策略如何兼容？ | 多因子优先，统一 StrategyAdapter 输出合同。 | 多因子专用 runner；每类策略各自 runner。 | 推荐方案同时满足多因子优先和通用扩展；专用 runner 后续会分裂。 | adapter contract 需要稳定测试。 | 若多因子字段不稳定，先降级 legacy adapter。 |
| DQ-CP5-CR091-04 | implementation | 首版产物形态是什么？ | `trading/strategy_runner` 薄模块 + 离线 checker + fixtures/tests。 | 只写脚本；直接完整 runtime。 | 推荐方案可复用且仍小范围；脚本方案扩展弱，runtime 方案越权。 | 实现量高于单脚本。 | 若范围过大，可先交付 adapters + evidence 子集。 |
| DQ-CP5-CR091-05 | security | runner 是否直接从 NAS 运行策略？ | 不直接运行；只读本地 immutable cache / active pointer。 | NAS 原地运行；自动 pull/publish。 | 推荐方案避免 NAS 和发布边界；备选需新 CR。 | 后续需要单独处理包分发。 | 用户要求 NAS 自动取包时启动 CR093。 |
| DQ-CP5-CR091-06 | runtime_authorization | 首版是否允许真实 QMT runtime？ | 自动验证只允许 fake transport；真实只读 smoke 需后续逐 run 授权。 | 立即连接 QMT；完全取消只读 smoke。 | 推荐方案保留验证路线但不越权。 | 无真实 runtime ready 结论。 | 用户给出逐 run 授权后再开 runtime gate。 |
| DQ-CP5-CR091-07 | follow_up_tracking | 下单 / 撤单是否纳入 CR091？ | 不纳入，拆为 CR092 候选。 | 纳入 CR091 后半段。 | 推荐方案隔离 order write 风险；纳入会扩大门禁复杂度。 | CR091 不能验证下单链路。 | 用户明确要求订单写验证时启动 CR092。 |
| DQ-CP5-CR091-08 | risk_acceptance | 是否接受 CR091 首版不证明真实可交易？ | 接受；只证明 runner 合同和离线验证。 | 要求真实交易机验证后才通过；取消 runner。 | 推荐方案符合当前权限；真实验证需要额外授权。 | CP8 仍可能是 READY_WITH_RISK。 | 用户拒绝风险时暂停或新增 runtime gate。 |

如果你回复 approve，表示你接受以上 8 项推荐方案，不表示授权以下 8 类不授权项：

不授权项：

1. QMT / MiniQMT / XtQuant / gateway / runner runtime 启动。
2. NAS read / write / list / copy / pull / publish。
3. 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
4. submit / cancel。
5. simulation / live。
6. provider / lake / catalog / publish。
7. 恢复 CR020 的用户删除 QMT gateway 路线。
8. 自动启动 CR089 runtime。

推荐回复之一：

approve
修改: <具体修改点>
reject
