---
feature_id: "FEAT-12"
feature_name: "QMT Gateway Service Layer"
status: "ready-for-cp5-review"
version: "1.0"
change: "CR-138"
created_at: "2026-06-24T16:10:00+08:00"
created_by: "host-orchestrator"
---

# FEAT-12 QMT Gateway Service Layer Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-24 | host-orchestrator | 初版测试计划，覆盖 REST-only P0、查询服务、订阅、订单回报、恢复、审计和配置变更 |

## 测试策略

| 层级 | 范围 | 默认方式 | 禁止项 |
|---|---|---|---|
| 静态 guardrail | runtime auth、secret redaction、protocol boundary | pytest / script | 不启动 Gateway、不读 `.env` |
| 合同测试 | GatewayHealth、CapabilitySnapshot、QueryResult、GatewayCommand | fixture | 不连接 QMT |
| REST P0 测试 | route contract / response shape | fake app / service object | 不绑定真实端口 |
| 查询测试 | calendar / commission / PnL unavailable / blocked | fixture | 不读真实账户 |
| 订阅 / 订单测试 | subscription state、hard reject、execution report fixture | fake adapter | 不 submit/cancel |
| 恢复 / 审计测试 | degraded / unknown / audit search | event replay fixture | 不使用原始日志 |

## 测试用例矩阵

| 测试 ID | Story | 场景 | 预期 |
|---|---|---|---|
| FEAT12-T01 | CR138-S05 | health / capabilities without runtime | 返回 static / unavailable，不升级授权 |
| FEAT12-T02 | CR138-S05 | REST-only route registry | SSE/WebSocket/gRPC/FIX route count=0 |
| FEAT12-T03 | CR138-S06 | 本地交易日历可用 | TradingCalendar.available |
| FEAT12-T04 | CR138-S06 | 账户级 PnL 无授权 | PnLSnapshot.blocked |
| FEAT12-T05 | CR138-S06 | QMT 不支持佣金查询 | CommissionSchedule.unavailable_with_reason |
| FEAT12-T06 | CR138-S07 | 行情订阅无 market scope | MarketSubscription.blocked |
| FEAT12-T07 | CR138-S07 | order submit 无授权 | GatewayCommand.hard_rejected，adapter_calls=0 |
| FEAT12-T08 | CR138-S07 | execution report unknown | RecoveryPlan.manual_takeover |
| FEAT12-T09 | CR138-S08 | audit search with sensitive fields | redacted / blocked |
| FEAT12-T10 | CR138-S08 | config change missing rollback | GatewayChangePlan.rejected |

## 验收命令候选

| 命令 | 触发 Story | 说明 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr138_qmt_gateway_service_layer.py` | CR138-S05..S07 | 后续实现后运行 |
| `uv run --python 3.11 python -m py_compile trading/qmt_gateway_contracts.py trading/qmt_gateway_service.py` | CR138-S05..S07 | 后续实现后运行 |
| `uv run --python 3.11 pytest -q tests/test_cr019_qmt_gateway_lifecycle.py tests/test_cr020_server_qmt_login_session.py` | 回归候选 | 确保不破坏历史 Gateway 边界 |

## 手工 / 授权验证

默认 N/A。任何真实 QMT / MiniQMT / XtQuant、账户、行情、订单、submit/cancel、simulation/live 验证必须先通过 runtime_authorization gate。

