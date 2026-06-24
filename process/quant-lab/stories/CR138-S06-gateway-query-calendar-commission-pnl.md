---
story_id: "CR138-S06-gateway-query-calendar-commission-pnl"
cr_id: "CR-138"
title: "Gateway query service for calendar, commission, and PnL"
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
    - "query authorization"
    - "account data redaction"
    - "fee and PnL model"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S06 Gateway Query Service for Calendar, Commission, and PnL

## 用户价值

用户可以在盘前和盘后获得交易日历、交易窗口、费用估算和收益摘要，同时清楚哪些结果来自本地参考、配置估算或授权账户查询。

## 范围

- TradingCalendar、TradingWindow。
- CommissionSchedule、CostEstimate。
- PnLSnapshot、ReturnSummary。
- account_readonly authorization 与 redaction。

## 非范围

- 不读取真实账户，除非后续单独 runtime_authorization。
- 不伪造 broker confirmed 佣金或 PnL。
- 不写 broker lake。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | calendar local source 缺失时返回 unavailable，不推断交易日。 |
| AC-02 | commission source 必须是 configured / broker_confirmed / estimated / unavailable 之一。 |
| AC-03 | account-level PnL 无授权时 blocked。 |
| AC-04 | QMT 不支持查询时返回 unavailable_with_reason。 |

## 技术说明

需要 full LLD，重点说明本地参考数据、账户只读授权、脱敏、estimated 与 broker_confirmed 的边界。

