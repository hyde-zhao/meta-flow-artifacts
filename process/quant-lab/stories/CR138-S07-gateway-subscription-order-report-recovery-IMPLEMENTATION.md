---
story_id: "CR138-S07-gateway-subscription-order-report-recovery"
status: "implemented"
owner: "host-orchestrator"
implemented_at: "2026-06-24T16:17:40+08:00"
cp6: "PASS"
---

# CR138-S07 Implementation

## 实现对象

| 对象 | 路径 | 说明 |
|---|---|---|
| Gateway gates | `trading/qmt_gateway_gates.py` | `gate_market_subscription`、`hard_reject_gateway_command`。 |
| GatewayService handlers | `trading/qmt_gateway_service.py` | `register_subscription`、`pull_subscription_events`、`validate_gateway_command`、`ingest_execution_report`、`build_recovery_plan`。 |
| Contracts | `trading/qmt_gateway_contracts.py` | MarketSubscription、GatewayCommand、GatewayCommandDecision、GatewayEvent、ExecutionReport、RecoveryPlan。 |
| Tests | `tests/test_cr138_gateway_subscription_order_report_recovery.py` | subscription scope、submit/cancel hard reject、REST pull events、manual recovery。 |

## 设计契约映射

| LLD 契约 | 实现位置 | 验证 |
|---|---|---|
| market_readonly 缺失 blocked | `gate_market_subscription()` | `test_market_subscription_blocks_without_scope_and_never_calls_adapter` |
| submit/cancel 缺授权 hard_rejected | `hard_reject_gateway_command()` | `test_gateway_command_hard_rejects_submit_cancel_without_authorization` |
| P0 subscription REST pull | `register_subscription()` / `pull_subscription_events()` | `test_gateway_service_subscription_events_are_rest_pull_fixture_only` |
| unknown report -> manual recovery | `ExecutionReport` / `build_recovery_plan()` | `test_execution_report_unknown_requires_manual_takeover_and_recovery_is_manual_only` |

## 验证结果

CR138 定向测试 PASS：48 passed。

## 不授权边界

`hard_rejected` 是本地拒绝，不发送到 broker；RecoveryPlan 不自动 retry、不自动解除 kill switch。
