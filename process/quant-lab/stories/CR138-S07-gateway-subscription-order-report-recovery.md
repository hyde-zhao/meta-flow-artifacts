---
story_id: "CR138-S07-gateway-subscription-order-report-recovery"
cr_id: "CR-138"
title: "Gateway subscription, order report, and recovery boundary"
status: "lld-ready-for-review"
priority: "P0"
wave: "CR138-W3-OPERATIONAL-FLOWS"
depends_on:
  - "CR138-S01-shared-contracts-authorization-audit"
  - "CR138-S05-gateway-lifecycle-health-rest-contract"
dependency_type:
  - "contract"
  - "runtime"
feature_design_refs:
  - "process/docs/features/qmt-gateway-service-layer/DESIGN.md"
  - "process/docs/features/qmt-trading-governance/DESIGN.md"
  - "process/docs/features/runtime-authorization-safety/DESIGN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "market subscription"
    - "order command boundary"
    - "recovery path"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S07 Gateway Subscription, Order Report, and Recovery Boundary

## 用户价值

用户可以理解行情订阅、订单命令、执行回报和故障恢复如何被 Gateway 管理，同时确保无授权时不会触发真实市场或订单动作。

## 范围

- MarketSubscription、GatewayCommand、ExecutionReport。
- order hard reject / future runtime send boundary。
- RecoveryPlan、manual_takeover、kill switch 交互。

## 非范围

- 不订阅真实行情。
- 不提交或撤销真实订单。
- 不自动恢复 QMT session。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | market_readonly 未授权时 subscription blocked。 |
| AC-02 | submit/cancel 未授权时 GatewayCommand.hard_rejected，adapter_calls=0。 |
| AC-03 | ExecutionReport 支持 unknown / stale。 |
| AC-04 | Gateway degraded 时不自动重发订单，不自动解除 kill switch。 |

## 技术说明

需要 full LLD，重点说明 subscription state、order scope、execution report、recovery supervisor 和 fail-closed 行为。

