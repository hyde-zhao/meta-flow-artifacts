---
story_id: "CR138-S06-gateway-query-calendar-commission-pnl"
status: "implemented"
owner: "host-orchestrator"
implemented_at: "2026-06-24T16:17:40+08:00"
cp6: "PASS"
---

# CR138-S06 Implementation

## 实现对象

| 对象 | 路径 | 说明 |
|---|---|---|
| Query handlers | `trading/qmt_gateway_service.py` | `query_trading_calendar`、`query_commission_schedule`、`estimate_cost`、`query_pnl_snapshot`、`query_return_summary`。 |
| Query contracts | `trading/qmt_gateway_contracts.py` | TradingCalendar、CommissionSchedule、CostEstimate、PnLSnapshot、ReturnSummary。 |
| Tests | `tests/test_cr138_gateway_query_calendar_commission_pnl.py` | local calendar、missing no inference、commission source、PnL auth/unavailable。 |

## 设计契约映射

| LLD 契约 | 实现位置 | 验证 |
|---|---|---|
| calendar 本地参考，缺失不推断 | `query_trading_calendar()` | `test_trading_calendar_uses_local_reference_and_never_infers_missing_days` |
| commission source 必填 | `query_commission_schedule()` | `test_commission_schedule_and_cost_estimate_are_source_tagged_not_broker_fact` |
| broker_confirmed 需要 account_readonly | `query_commission_schedule()` | `test_broker_confirmed_commission_requires_account_readonly_authorization` |
| PnL 缺 auth blocked / unsupported tagged | `query_pnl_snapshot()` | `test_pnl_query_blocks_without_account_auth_and_reports_unsupported_with_reason` |

## 验证结果

CR138 定向测试 PASS：48 passed。

## 不授权边界

所有账户级结果只返回 redacted / unavailable / blocked 摘要；未查询真实账户、资金、持仓、委托、成交或 broker payload。
