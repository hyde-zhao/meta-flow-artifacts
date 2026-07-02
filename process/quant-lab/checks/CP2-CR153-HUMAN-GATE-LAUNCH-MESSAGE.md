# CP2 CR153 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR153-EVENT-DRIVEN-STRATEGY-E2E-SCOPE.md`

自动预检结论：PASS

上下文胶囊：`process/context/CP2-CR153-EVENT-DRIVEN-STRATEGY-E2E-SCOPE-CONTEXT.yaml`（read_profile=compact）

Context Capsule Summary:

- capsule 路径：`process/context/CP2-CR153-EVENT-DRIVEN-STRATEGY-E2E-SCOPE-CONTEXT.yaml`
- capsule 状态：ready
- read_profile：compact

审批者摘要：

- 本次确认服务的整体目标：确认 CR153 Event-Driven Strategy E2E Framework Foundation 的 CP2 范围基线。
- 推荐动作：`approve`，接受 9 项推荐方案。
- approve 后会发生什么：CR153 进入 CP3 设计；不会进入 Story、LLD、实现或运行时。
- approve 不授权什么：不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog/event-store/model-registry 写入；不授权 live event listener、paper OMS、broker adapter、真实事件 feed、真实下单、真实数据验证或源码实现。
- 不确认会阻塞什么：阻塞 CR153 的 CP3 设计、Story 拆解、LLD、实现、验证和 release readiness。

本轮待人工决策项：9

决策收集覆盖：已扫描 10 个来源，发现候选问题 53 个，纳入待决策 9 个；已补 CR153 专属 SGQ 证据 `process/discussions/CP2-CR153-SCENARIO-DISCUSSION-LOG.md` / `process/checks/CP2-CR153-DISCUSSION-CHECKPOINT.json`；N/A / 缺失来源 1 个：`docs/product/USE-CASES.md` 缺失，已使用 `process/USE-CASES.md` 中已确认 UC-60 作为 fallback。

决策分层：

- 必须用户决策：9
- 高风险策略确认：1
- agent 默认处理：3
- 仅审计记录：6

如果你回复 approve，表示你接受以下 9 项推荐方案。

流程补证据说明：CR153 standard CP2 的 Scenario Gray Areas / SGQ 讨论证据已补齐；这不改变 9 项推荐方案，也不扩大 CR153 范围。

不表示授权：真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog/event-store/model-registry 写入、live event listener、paper OMS、broker adapter、真实事件 feed、真实下单、真实数据验证或源码实现。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR153-EVENT-TIME-SEMANTICS | architecture | 如何定义和校验 `event_time`、`available_at`、`decision_time`？ | 三时间全部必填或 `N/A-with-reason`；只消费 `available_at <= decision_time` 的事件事实。 | 只要求部分时间；允许自动推断；暂不建 gate。 | 推荐方案防 lookahead；备选会削弱事件 PIT 语义。 | 影响 EventResearchSpec、PIT gate、replay、admission。 |
| DQ-CP2-CR153-EVENT-STUDY-METHOD | implementation | first wave 采用哪些事件研究方法契约？ | 定义 estimation window、event window、normal return model、CAR/BHAR slots 和 calendar-time slots；fixture 只验证 contract semantics。 | 只做 forward return；完整统计库；只写文档。 | 推荐方案覆盖方法论且范围可控。 | 影响 EventStudyMethodSpec 和 CP5 Story 边界。 |
| DQ-CP2-CR153-TEST-FAMILY | implementation | 事件研究检验族如何进入 admission？ | 预留 Patell / BMP / generalized sign / rank / bootstrap slots、report refs 和 status；不实现完整统计库。 | 不预留；完整实现；只支持普通 t-test。 | 推荐方案保留扩展位，避免范围膨胀。 | 影响 EventStudyTestReport 和 Event admission gate。 |
| DQ-CP2-CR153-CLUSTER-ENDOGENEITY | architecture | 如何表示 overlapping events、clustering 和 endogeneity？ | 定义 overlap / cluster report slots 和 endogeneity treatment note；unsupported methods 显式 `NEEDS_REVIEW`、`BLOCKED` 或 `n/a-with-reason`。 | 忽略；完整算法；自由文本备注。 | 推荐方案可审计且范围可控。 | 影响 admission fail-closed 语义和风险报告。 |
| DQ-CP2-CR153-GATE-RELATION | architecture | Event admission gate 与 CR151 / CR152 gates 的关系是什么？ | 新建 Event-specific admission gate 和 adapter，复用四态 status、blocked reasons 和 admission package linkage。 | 塞入 CR151；复用 CR152；完全独立。 | 推荐方案职责清晰且状态一致。 | 影响 StrategyAdmissionPackage 和 CR154 governance。 |
| DQ-CP2-CR153-EVENT-TO-ORDER-BOUNDARY | security | event-to-order trace 是否授权 runtime、order 或 broker access？ | 不授权。只定义 event -> signal -> target/order-intent trace contract。 | 实现 paper OMS；readonly feed；broker adapter。 | 推荐方案保留审计链但不越权。 | 高风险边界；防止 CR153 变运行时 CR。 |
| DQ-CP2-CR153-CV-STRATEGY | architecture | EV-GAP-8 walk-forward / OOS / purged-embargo CV 如何处理？ | 只预留 event CV slot 和 split audit refs；完整框架由 CR154 横切治理承接。 | 自建 Event CV；不记录；复用 ML CV 默认。 | 推荐方案避免重复框架并保留 blocker 位。 | 影响 CR154 范围和 EventResearchSpec 可扩展性。 |
| DQ-CP2-CR153-SURVIVORSHIP-SLOT | architecture | EV-GAP-9 survivorship bias 是否进入 first wave？ | 在 `EventResearchSpec` 预留 `universe_pit_audit` slot；完整 gate 归入 CR154。 | 完整实现；不记录；只写备注。 | 推荐方案机器可见且不挤占 first wave。 | 影响 universe PIT / event sample validity。 |
| DQ-CP2-CR153-METHOD-SLOT-ONLY | implementation | first wave 是否实现事件检验族、cluster/endogeneity 具体算法？ | 否。只定义 contract slot、status、refs 和 `n/a-with-reason`。 | 完整实现算法；删除 slots；引入外部统计框架。 | 推荐方案与 CR152 slot-only 风格一致，范围可控。 | 防止 first wave 膨胀。 |

不授权项：

- real lake read/write、NAS、provider、credential / `.env` read
- QMT / MiniQMT / xtquant / gateway runtime、simulation、paper、live、trading
- live event listener、broker read/write/submit/cancel、真实账户查询
- external framework clone / install / run、Git remote write
- catalog pointer mutation、feature/label/event/model/prediction store write
- real event feed execution、real order flow、real data validation

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

可直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```
