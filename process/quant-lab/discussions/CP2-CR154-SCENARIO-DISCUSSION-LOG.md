---
discussion_id: "CP2-CR154-SCENARIO-DISCUSSION"
cr_id: "CR-154"
status: "completed"
created_at: "2026-07-02T23:25:00+08:00"
owner: "host-orchestrator"
source: "user-startup-request"
---

# CP2 CR154 Scenario Discussion Log

## Scenario Gray Areas

| SGQ ID | 问题 | 用户回答 / 当前明确约束 | 复述确认 | 影响面 |
|---|---|---|---|---|
| SGQ-CR154-001 | CR154 是否应作为独立 formal CR 启动，还是继续作为 CR153 deferred note？ | 用户明确要求“启动 CR154 Cross-Strategy Production Reliability Gates”，并要求规则 41 预检、CR-154、CP0、CP1、CP2。 | CR154 应单独建正式 CR，不复用 CR153 CP8 approval，也不把 CR153 deferred note 当成已批准实现范围。 | CR lifecycle、CP0/CP1/CP2、CR-INDEX、STATE。 |
| SGQ-CR154-002 | CR154 first wave 覆盖哪些横切生产可靠性 gate？ | 用户列出 backtest trap gate、walk-forward/OOS/purged-embargo、survivorship-free / PIT universe、capacity/market impact/liquidity、regime/attribution/reconciliation slots、CR151/152/153 admission 默认强制策略。 | CP2 待决策清单按 7 项覆盖，不新增真实 runtime 或真实数据范围。 | Scope、architecture、implementation。 |
| SGQ-CR154-003 | CR154 是否授权真实 lake/NAS/provider/runtime/broker/credential/feed/order/reconciliation/publish？ | 用户明确要求保持 local/static/fixture-only，除非另开 runtime authorization gate；不得连接真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework。 | 本轮 CP2 approve 只允许进入 CP3 设计，不授权任何真实外部系统、数据、runtime、交易、registry/store/catalog 写入或真实 reconciliation。 | Security、runtime_authorization、risk acceptance。 |
| SGQ-CR154-004 | `FU-CR152-001` 是否会阻塞 CR154？ | 用户明确说明 `FU-CR152-001` test taxonomy / provenance hygiene 保持 candidate，不占执行锁。 | CR154 不启动或处理 `FU-CR152-001`；后者仍为 backlog candidate。 | CR conflict precheck、file ownership、follow-up tracking。 |
| SGQ-CR154-005 | CR154 与 CR151/CR152/CR153 的关系如何处理？ | 用户要求确认 CR154 不与 CR151/CR152/CR153 源码 / Story / file owner 产生未授权重叠，并推荐 admission gate 默认强制策略作为 CR154 范围。 | CP2 只确认跨策略 gate policy 目标；后续 CP3/CP4 才冻结文件 owner / Story 边界。 | Architecture、story planning、file owner。 |

## 结论

- SGQ 状态：completed
- 阻断问题：0
- 进入 CP2 待人工决策项：7
- 不授权项：真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework、live listener、catalog/event store/model registry、真实 feed、真实下单、真实数据验证、真实 reconciliation、真实发布执行。
