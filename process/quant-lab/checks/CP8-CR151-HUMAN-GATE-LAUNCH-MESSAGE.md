请审查：process/checkpoints/CP8-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-RELEASE-READINESS.md

自动预检结论：PASS / READY_WITH_RISK pending approval

Context Capsule：process/release/RELEASE-CONTEXT-CR151.yaml（read_profile=compact；minimal release profile）

审批者摘要：
- 本次确认服务的整体目标：完成 CR151 multifactor strategy E2E statistical admission framework 的 CP8 release readiness，决定是否接受 STATE v2 hygiene 作为过程 caveat，并按 local/static/fixture 范围收尾。
- 推荐动作：approve，接受 `DEC-CR151-CP8-001`，将 CR151 CP8 release decision 标记为 `READY_WITH_RISK`。
- approve 后会发生什么：记录用户接受 STATE v2 hygiene caveat；CR151 本地/static/fixture 多因子统计准入框架按 `READY_WITH_RISK` 收尾；后续进入 CR152/CR153/CR154 或 STATE hygiene follow-up。
- approve 不授权什么：不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/Git remote/外部框架运行，不授权 publish、push、依赖安装、catalog pointer mutation、STATE slimming 执行或任何外部写入。
- 不确认会阻塞什么：阻塞 CR151 CP8 closure；若用户不接受风险，则先回到 STATE v2 hygiene rework，再重新进入 CP8。

本轮待人工决策项：1

决策收集覆盖：已扫描 7 个来源，纳入待决策 1 个；R01/R02 已关闭，STATE v2 hygiene 是唯一开放风险。

决策分层：
- 必须用户决策：1
- 高风险策略确认：0
- agent 默认处理：3
- 仅审计记录：1

如果你回复 approve，表示你接受以下 1 项推荐方案，不表示授权以下禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DEC-CR151-CP8-001 | risk_acceptance | 是否接受 CR151 在 STATE v2 hygiene 未立即整改的情况下按 `READY_WITH_RISK` 收尾？ | 接受本次 process caveat，将 CR151 按 local/static/fixture 范围标记为 `READY_WITH_RISK`；STATE v2 slimming 另起 follow-up。 | A: 不接受风险，先执行 STATE v2 hygiene rework，再重跑 CP8；B: 将 STATE v2 hygiene 并入 CR151 当前范围并阻塞所有后续策略框架推进。 | 推荐方案避免把策略框架交付扩大成 Meta Flow 状态迁移；A 最严格但延迟 closure；B 范围膨胀最大。 | 接受后残余风险是过程状态入口仍不满足 v2 slim contract；不影响 CR151 源码、测试、static-only 证据或授权边界。 |

CP8 evidence clarifications：
- `effective_validation_mode=static-only`；不代表 runtime evidence。
- `CR151-CP7-R01` 已关闭：canonical `docs/quality/TEST-STRATEGY.md` 已补齐。
- `CR151-CP7-R02` 已关闭：4 个 CP7 verify packet 已补齐 acceptance / plan / refs。
- `CR151-CP8-R03-STATE-V2-HYGIENE` 是唯一开放 caveat。

不授权项：
- real lake read or write
- NAS sync/write/restore/metadata normalization
- provider fetch
- catalog pointer mutation
- credential or `.env` read
- QMT, MiniQMT, xtquant or gateway runtime
- simulation, paper, live, trading or broker operation
- Git remote write
- external framework clone, install or run
- STATE slimming execution

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

审查后请在“人工审查结果”中填写结论，也可以直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```
