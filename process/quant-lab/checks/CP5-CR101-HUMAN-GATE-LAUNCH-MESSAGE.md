请审查：/home/hyde/workspace/quant-lab/process/checkpoints/CP5-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-LLD-BATCH.md

自动预检结论：PASS。CP4 Story DAG / 并行安全 PASS；4 个 CP5 LLD implementability check 均 PASS；阻断项 0。

Context Capsule：`process/context/CP5-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。4 个 Story 已进入 `lld-ready-for-review`，CP5 批准前不得实现。

决策收集覆盖摘要：已扫描 CP3 review、Story backlog、Development Plan、S01-S04 LLD、CP4/CP5 auto checks、当前 runtime/security boundary；候选问题 33 个，纳入待人工决策 5 个，未回答阻断项 0。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 7 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR101-01 | implementation | 是否批准 4 个 Story 的 LLD batch 和 Wave / merge order？ | 批准 S01 -> S02/S03 -> S04；CP5 后才允许受限离线实现。 | 拆成两个 CP5 批次；退回 CP4 重拆。 | 推荐方案覆盖完整且依赖清晰；备选更保守但延长门禁。 | 影响 story execution、文件 owner、CP6/CP7 验证范围。 |
| DQ-CP5-CR101-02 | implementation | manifest target taxonomy 是否按 S01 LLD 冻结？ | 冻结 target + adapter 双边界，当前唯一 implemented delivery target 为 `qmt_terminal_direct`。 | QMT-only schema；保留旧 `miniqmt_runner` delivery target 兼容。 | 推荐方案符合 CP3；备选会返工或继续混淆。 | 影响 package_loader、manifest fixture、package exchange、adapter payload 和 tests。 |
| DQ-CP5-CR101-03 | implementation | checker / fixture / evidence 负向验证是否按 S02/S03 LLD 执行？ | 执行 8 类 checker fail-closed、evidence sensitive hit 0、forbidden counters 0、readonly gateway regression。 | 只覆盖 checker；只改 evidence。 | 推荐方案关闭主要漂移；备选留下边界缺口。 | 影响 package_exchange、adapters、evidence、readonly_gateway 和 tests。 |
| DQ-CP5-CR101-04 | runtime_authorization | CP5 approve 是否授权 NAS、凭据、QMT/MiniQMT/XtQuant/gateway runtime、simulation/live、交易或 publish？ | 不授权；只允许后续离线代码 / 文档实现和本地测试。 | 另起 per-run gate；授权 agent 代跑真实 runtime。 | 推荐方案权限最小；备选需要独立高风险门禁。 | 不证明真实 NAS/QMT/MiniQMT/gateway/account/trading ready。 |
| DQ-CP5-CR101-05 | risk_acceptance | 是否接受 CP5 后仍只能达到 offline READY_WITH_RISK 的路线？ | 接受，CP7/CP8 以离线 contract / checker / redaction 证据关闭，真实验证后置。 | 要求真实验证后才继续；关闭 CR101 为 blocked。 | 推荐方案先消除架构漂移；备选引入高权限或无法修复当前混淆。 | 残余风险是离线 evidence 不能代表真实 runtime / NAS / account / trading 链路。 |

不授权项：

- 不访问、列取、读取、复制、写入、挂载、发布、拉取或删除真实 NAS。
- 不读取 `.env`、token、secret、API key、password、private key、QMT 凭据或任何凭据。
- 不读取账户、资金、持仓、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不自动启动 CR089、CR020 恢复或任何真实验证 follow-up。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

审查后请在“人工审查结果”中填写结论，也可以直接回复以下任一整行：
approve
修改: <具体修改点>
reject
