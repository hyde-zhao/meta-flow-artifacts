# CR138 Runner / QMT Gateway Fixture Test Matrix

## Scope

本矩阵覆盖 CR138 S01..S08 的本地 fixture / contract 验证。当前矩阵不授权真实 runtime、QMT、MiniQMT、XtQuant、gateway 启动、端口绑定、凭据读取、账户 / 行情 / 订单真实查询、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。

## Story Coverage

| Story | Scenario | Fixture / Test | Assertion | No-real-operation evidence |
|---|---|---|---|---|
| CR138-S01 | shared authorization and audit | `tests/test_cr138_shared_contracts_authorization_audit.py` | missing scope blocked, audit identity required | forbidden counters all zero |
| CR138-S02 | runner plan and preflight | `tests/test_cr138_runner_plan_preflight_control.py` | missing auth/data blocked, batch preflight local | adapter_calls=0 |
| CR138-S03 | event, rebalance, ops summary | `tests/test_cr138_runner_event_signal_rebalance_tracking.py` | duplicate event skipped, draft only | order_submit=0, order_cancel=0 |
| CR138-S04 | evidence, review, incident | `tests/test_cr138_runner_evidence_review_incident_lifecycle.py` | raw refs blocked, rollback required | raw copy count=0 |
| CR138-S05 | gateway lifecycle and REST | `tests/test_cr138_gateway_lifecycle_health_rest_contract.py` | REST-only, no port bind, change dry-run | gateway_start=0, port_bind=0 |
| CR138-S06 | calendar, commission, PnL | `tests/test_cr138_gateway_query_calendar_commission_pnl.py` | calendar no inference, PnL auth-gated | account_query=0 |
| CR138-S07 | subscription, order report, recovery | `tests/test_cr138_gateway_subscription_order_report_recovery.py` | market/order scopes fail closed | order_submit=0, order_cancel=0 |
| CR138-S08 | docs and authorization runbook | `tests/test_cr138_docs_fixtures_authorization_runbook.py` | no authorization claims, template complete | runtime_authorization required |

## Forbidden Operations

| Operation | Expected Count |
|---|---:|
| runtime_start | 0 |
| gateway_start | 0 |
| port_bind | 0 |
| credential_read | 0 |
| qmt_operation | 0 |
| xtquant_import | 0 |
| account_query | 0 |
| market_query | 0 |
| order_query | 0 |
| order_submit | 0 |
| order_cancel | 0 |
| simulation_run | 0 |
| live_run | 0 |
| nas_access | 0 |
| provider_fetch | 0 |
| lake_write | 0 |
| catalog_publish | 0 |
| git_remote_write | 0 |
| adapter_calls | 0 |

## CP7 Boundary

CP7 若仅执行本矩阵和静态 / fixture 测试，只能声明 `fixture/static verified`，不得声明真实运行、真实 QMT 或真实 broker 已验证。任何真实验证必须先发起独立 runtime_authorization gate。
