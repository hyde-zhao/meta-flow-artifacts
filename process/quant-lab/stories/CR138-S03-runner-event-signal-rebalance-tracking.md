---
story_id: "CR138-S03-runner-event-signal-rebalance-tracking"
cr_id: "CR-138"
title: "Runner event, signal, rebalance, run tracking, and ops summary"
status: "lld-ready-for-review"
priority: "P0"
wave: "CR138-W3-OPERATIONAL-FLOWS"
depends_on:
  - "CR138-S01-shared-contracts-authorization-audit"
  - "CR138-S02-runner-plan-preflight-control"
dependency_type:
  - "contract"
  - "runtime"
feature_design_refs:
  - "process/docs/features/runner-control-plane/DESIGN.md"
  - "process/docs/features/qmt-trading-governance/DESIGN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "event idempotency"
    - "OMS risk handoff"
    - "run state tracking"
    - "ops dashboard cli summary"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S03 Runner Event, Signal, Rebalance, Run Tracking, and Ops Summary

## 用户价值

用户可以把事件 / 信号 / 再平衡意图纳入 Runner 运营视图，并通过本地 CLI summary 查看运行状态、风险阻断和 no-real-operation counters，而不是让信号直接变成真实订单。

## 范围

- event / signal schema 与幂等。
- Rebalance planner 到 OrderIntentDraft / OrderIntent 的合同。
- RunState 消费 GatewayEvent / ExecutionReport 摘要。
- OpsSummary / CLI Summary 本地只读渲染。
- BatchOpsSummary 本地批量运行汇总，只消费 S02 RunPlanBatch 和 CR137 registry refs。

## 非范围

- 不提交真实订单。
- 不绕过 FEAT-06 风控。
- 不连接 QMT 或订阅真实行情。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | 重复 event_id / idempotency_key 返回 duplicate / skip。 |
| AC-02 | risk fail 或 no auth 时 order adapter call 为 0。 |
| AC-03 | RunState 支持 degraded / paused / manual_takeover。 |
| AC-04 | Runner 只消费 redacted Gateway event / report，不消费原始账户日志。 |
| AC-05 | CLI summary 只读取本地 RunState / redacted refs / no-real-operation counters，runtime_calls=0。 |
| AC-06 | BatchOpsSummary 只读取本地 batch plan / RunState / registry refs，runtime_calls=0。 |

## 技术说明

需要 full LLD，重点说明事件顺序、幂等、risk handoff、状态汇总、OpsSummary / CLI Summary、BatchOpsSummary 和 runtime dependency 的开发门控。
