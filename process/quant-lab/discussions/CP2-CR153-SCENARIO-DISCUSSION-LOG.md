---
discussion_id: "CP2-CR153-SCENARIO-DISCUSSION"
status: "completed-after-cp2-approval"
created_at: "2026-07-02T16:45:39+08:00"
created_by: "host-orchestrator"
cr_id: "CR-153"
delegated_phase: "requirement-clarification"
interaction_mode: "current-review-plus-live-sgq"
source_context: "process/context/CP2-CR153-EVENT-DRIVEN-STRATEGY-E2E-SCOPE-CONTEXT.yaml"
discussion_checkpoint: "process/checks/CP2-CR153-DISCUSSION-CHECKPOINT.json"
cp2_final_human_gate_status: "approved"
cp2_approved_at: "2026-07-02T17:00:00+08:00"
---

# CP2 Scenario Discussion Log - CR153 Event-Driven Strategy E2E Framework Foundation

## 讨论目标

补齐 CR153 standard CP2 的 Scenario Gray Areas / SGQ 审计证据，确认本轮只推进 Event-Driven Strategy E2E Framework Foundation 的本地、静态、fixture-only 方法论骨架，不扩大到真实事件 feed、runtime、paper/live、broker、credential、lake/NAS/provider、catalog/event store/model registry 或真实下单。

本日志基于用户在 CP2 审查中的明确评审意见记录：内容方向可以支持进入 CP3，9 项推荐方案技术上建议接受；流程上应先补 CR153 专属 SGQ 讨论证据，再继续 CP2 approve。该意见不改变 CR153 scope，也不改变 9 个 DQ 的推荐结论；CP2 已于 `2026-07-02T17:00:00+08:00` 由用户回复 `approve继续推进项目` 批准，本日志作为补证闭环证据保留。

## Scenario Gray Areas

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 / 评审意见来源 | 状态 |
|---|---|---|---|---|---|
| SGA-CR153-01 | 三时间语义是否必须进入 first wave | 事件策略最容易把 `event_time` 当作可交易时间，若缺少 `available_at <= decision_time` 约束会引入 lookahead | EventResearchSpec / PIT gate / replay / admission / trace | 用户评审建议接受三时间必填或 N/A-with-reason，且 `available_at > decision_time` 必须 fail-closed | selected; resolved-by-user-review |
| SGA-CR153-02 | 事件研究方法应是 slot-only 契约还是完整统计库 | CR153 first wave 需要补方法论骨架，但完整 Patell/BMP/bootstrap/cluster robust/PSM/IV 会扩大范围 | EventStudyMethodSpec / EventStudyTestReport / CP5 Story 边界 | 用户评审建议定义 estimation window、event window、normal return model、CAR/BHAR/calendar-time slots，不实现完整统计库 | selected; resolved-by-user-review |
| SGA-CR153-03 | test family、cluster/endogeneity 如何可审计 | 只做普通 t-test 或自由文本会丢失 UC-60 和 E2E review 中的关键事件研究风险 | Event admission gate / fail-closed reasons / risk report | 用户评审建议 Patell/BMP/sign/rank/bootstrap、overlap、cluster、endogeneity 先进入 slots、status、refs、sample count、adjusted p-value 或 n/a-with-reason | selected; resolved-by-user-review |
| SGA-CR153-04 | Event gate 与 CR151 / CR152 gate 的职责关系 | 直接塞进 CR151/CR152 会污染多因子/ML gate；完全独立又会破坏四态 admission 一致性 | StrategyAdmissionPackage / gate adapter / completion map / CR154 governance | 用户评审建议新建 Event-specific gate + adapter，复用四态 status 和既有 admission package linkage | selected; resolved-by-user-review |
| SGA-CR153-05 | event-to-order trace 是否授权 runtime / order / broker | UC-60 完整生产能力包含 paper/live/order/reconciliation，但 CR153 first wave 只应证明静态 trace contract | security boundary / runtime authorization / broker boundary / CP3 scope | 用户评审强烈建议只做 event -> signal -> target/order-intent trace，不授权 paper OMS、broker、live listener 或真实 feed/order | selected; resolved-by-user-review |
| SGA-CR153-06 | CV 与 survivorship 是否本轮完整治理 | EV-GAP-8/9 是 blocker，但若在 CR153 完整实现会把 first wave 膨胀成横切回测治理 CR | CR154 split / EventResearchSpec extensibility / universe PIT audit | 用户评审建议 CR153 只预留 event CV slot、split audit refs 和 universe_pit_audit slot；完整治理归 CR154 | selected; resolved-by-user-review |

## SGQ 场景确认交互记录

### SGQ-CR153-001 - 三时间语义与 PIT fail-closed

- **问题**：CR153 是否必须把 `event_time`、`available_at`、`decision_time` 三时间语义作为 EventResearchSpec / PIT gate 的 first-wave 核心？
- **候选选项**：
  1. 三时间全部必填或 `N/A-with-reason`，策略决策只能消费 `available_at <= decision_time` 的事件事实；违反时 fail-closed（推荐）
  2. 只要求 `event_time` + `decision_time`
  3. 允许缺失 `available_at` 时自动推断
- **用户回答来源**：当前 CP2 审查意见明确建议接受推荐方案，并强调 `available_at > decision_time` 必须 fail-closed，不能自动推断通过。
- **复述确认**：CR153 CP3 应把三时间语义写入 HLD / ADR / failure path，不得把缺失 `available_at` 静默当作通过。
- **影响面**：EventResearchSpec、EventRevisionPITGate、event replay、admission、event-to-signal trace。

### SGQ-CR153-002 - 事件研究方法与 test family slot-only

- **问题**：CR153 first wave 应如何处理 estimation window、event window、normal return model、CAR/BHAR、calendar-time、Patell/BMP/sign/rank/bootstrap 等方法族？
- **候选选项**：
  1. 定义方法契约、test family slots、status、refs、sample count、adjusted p-value 位置；fixture 只验证 contract semantics，不实现完整统计库（推荐）
  2. 本轮完整实现事件研究统计库
  3. 只做普通 forward return 或普通 t-test
- **用户回答来源**：当前 CP2 审查意见建议接受推荐方案，并明确“CP3 可给默认枚举和字段，不实现完整统计库”“不要只支持普通 t-test”。
- **复述确认**：CR153 CP3 应设计方法 contract 与可审计 slot；Patell/BMP/bootstrap 等具体算法仍不进入 first wave 实现。
- **影响面**：EventStudyMethodSpec、EventStudyTestReport、admission evidence、CP5 Story 粒度。

### SGQ-CR153-003 - cluster / endogeneity / unsupported method 语义

- **问题**：overlapping events、clustering 和 endogeneity treatment 在 CR153 中应如何表达？
- **候选选项**：
  1. 定义结构化 overlap / cluster / endogeneity slots；unsupported methods 必须 `NEEDS_REVIEW`、`BLOCKED` 或 `n/a-with-reason`（推荐）
  2. 本轮忽略该类风险
  3. 只用自由文本备注
- **用户回答来源**：当前 CP2 审查意见建议接受推荐方案，明确不能用自由文本糊过去。
- **复述确认**：CR153 CP3 应把 unsupported / missing 方法的失败路径写清，避免 event admission gate 静默通过。
- **影响面**：Event admission gate、blocked reasons、risk report、fixture contract。

### SGQ-CR153-004 - Event gate 与既有 gate 关系

- **问题**：Event admission gate 是否应复用 CR151 / CR152 的 existing gate，还是新建 Event-specific gate？
- **候选选项**：
  1. 新建 Event-specific admission gate + adapter，复用 CR151/CR152 四态 status、blocked reasons 和 admission package linkage（推荐）
  2. 直接塞入 CR151 statistical gate
  3. 复用 CR152 ML gate
  4. 完全独立，不复用 admission package
- **用户回答来源**：当前 CP2 审查意见建议接受推荐方案，认为直接塞入 CR151 或 CR152 会污染职责。
- **复述确认**：CR153 CP3 应设计 Event-specific gate 和 adapter，不改变 CR151 / CR152 已有能力默认行为。
- **影响面**：StrategyAdmissionPackage、completion map、gate adapter、CR154 governance。

### SGQ-CR153-005 - event-to-order 非 runtime 边界

- **问题**：CR153 的 event-to-order trace 是否授权 paper OMS、broker、live listener 或真实事件 feed？
- **候选选项**：
  1. 不授权。只做 event -> signal -> target/order-intent trace contract（推荐）
  2. 同时实现 paper OMS
  3. 启动 readonly live event feed
  4. 接入 broker adapter
- **用户回答来源**：当前 CP2 审查意见强烈建议接受推荐方案，指出该边界是高风险安全门。
- **复述确认**：CR153 CP3 不得设计真实 lake/NAS/provider/QMT/runtime/broker/credential/live listener/real feed/real order flow；如未来需要，必须另起 runtime_authorization / broker gate。
- **影响面**：security boundary、runtime authorization、UC-60 split、release wording。

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-CR153-01 | 完整 Patell / BMP / generalized sign / rank / bootstrap 算法实现 | SGQ-CR153-002 | CR153 first wave 只定义 slots / status / refs；完整统计库超出本轮范围 | 后续 Event method CR 或 CR154 明确授权具体算法实现 |
| DEF-CR153-02 | cluster robust variance、portfolio calendar-time regression、PSM / IV / matching | SGQ-CR153-003 | 本轮只定义结构化 slot 与 unsupported 状态，不实现具体方法 | 用户扩大范围或后续方法治理 CR 启动 |
| DEF-CR153-03 | full walk-forward / OOS / purged-embargo event CV framework | SGA-CR153-06 | 作为横切治理归入 CR154，避免三类策略重复实现 CV | CR154 启动或 CR153 后续专门 Event CV CR |
| DEF-CR153-04 | survivorship-free universe gate 完整实现 | SGA-CR153-06 | CR153 只预留 `universe_pit_audit` slot；完整治理归 CR154 | CR154 backtest trap gate 启动 |
| DEF-CR153-05 | paper/live event listener、paper OMS、broker adapter、真实 order flow | SGQ-CR153-005 | 需要独立 runtime / broker / credential / trading authorization | 用户明确发起 runtime_authorization / broker gate |

## 讨论恢复点

`process/checks/CP2-CR153-DISCUSSION-CHECKPOINT.json`

## 结论

- CR153 内容方向和 9 项推荐方案可支持进入 CP3 设计。
- 本轮先补齐 CR153 专属 SGQ 证据；CP2 最终结论仍等待用户明确 `approve`。
- CR153 scope 不扩大，仍保持 local/static/fixture-only。
- 不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework、live event listener、paper OMS、broker adapter、真实事件 feed、真实下单、真实数据验证、catalog / event store / model registry 写入或源码实现。
