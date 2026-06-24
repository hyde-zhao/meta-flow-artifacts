---
story_id: "CR138-S01-shared-contracts-authorization-audit"
status: "implemented"
owner: "host-orchestrator"
implemented_at: "2026-06-24T16:17:40+08:00"
cp6: "PASS"
---

# CR138-S01 Implementation

## 实现对象

| 对象 | 路径 | 说明 |
|---|---|---|
| Runner shared contracts | `trading/runner_control_contracts.py` | AuthorizationRecord、AuditRecord、BlockedResult、RunPlan、Preflight、RunState、OpsSummary、no-real-operation counters。 |
| Gateway shared contracts | `trading/qmt_gateway_contracts.py` | GatewayHealth、CapabilitySnapshot、RestRouteSpec、TradingCalendar、CommissionSchedule、PnLSnapshot、GatewayCommand、ExecutionReport、RecoveryPlan。 |
| Contract tests | `tests/test_cr138_shared_contracts_authorization_audit.py` | 字段、scope fail-closed、audit identity、REST-only、forbidden import。 |

## 设计契约映射

| LLD 契约 | 实现位置 | 验证 |
|---|---|---|
| 缺 scope 返回 blocked 且 adapter_calls=0 | `require_scope()` | `test_require_scope_fails_closed_and_health_does_not_authorize_order_scope` |
| health 不提升为账户 / 交易授权 | `GatewayHealth.runtime_authorized=false` | 同上 |
| audit_id / request_id 必填校验 | `validate_contract_ids()` | `test_audit_identity_validation` |
| no-real-operation counters 覆盖 runtime/QMT/交易/NAS/provider/lake/Git | `FORBIDDEN_OPERATION_COUNTER_FIELDS` | `test_cr138_shared_contract_fields_and_zero_counters` |

## 验证结果

| 命令 | 结果 |
|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr138_*.py tests/test_cr132_process_artifact_hygiene.py` | PASS，48 passed |
| `PYTHONPYCACHEPREFIX=/tmp/cr138-pycompile uv run --python 3.11 python -m py_compile ...` | PASS |

## 不授权边界

未启动 runtime / gateway，未导入 `xtquant`，未读取凭据，未查询账户 / 行情 / 订单，未 submit/cancel，未访问 NAS / provider / lake / catalog / Git remote。

## 结论

S01 实现完成，CP6 PASS，可供 S02/S05 及后续 Story 消费。
