---
feature_id: "FEAT-12"
feature_name: "QMT Gateway Service Layer"
status: "ready-for-cp5-review"
version: "1.0"
change: "CR-138"
created_at: "2026-06-24T16:10:00+08:00"
created_by: "host-orchestrator"
---

# FEAT-12 QMT Gateway Service Layer Tasks

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-24 | host-orchestrator | 初版任务清单，供 CR138 Story LLD 拆分和 CP5 批量确认 |

## 任务清单

| TASK ID | Story | 输入 | 输出 | 文件范围 | 验证入口 |
|---|---|---|---|---|---|
| FEAT12-TASK-01 | CR138-S01 | HLD / ADR / Domain Map | Gateway event / command / audit shared contract | `trading/qmt_gateway_contracts.py` | contract fixture |
| FEAT12-TASK-02 | CR138-S05 | FEAT12-TASK-01 | GatewayHealth / CapabilitySnapshot | `trading/qmt_gateway_service.py` | lifecycle fixture |
| FEAT12-TASK-03 | CR138-S05 | REST-only ADR | REST P0 route registry | `trading/qmt_gateway_service.py` / CLI | route count fixture |
| FEAT12-TASK-04 | CR138-S06 | calendar local source | TradingCalendar / TradingWindow | `trading/qmt_gateway_service.py` | calendar fixture |
| FEAT12-TASK-05 | CR138-S06 | fee config / account auth | CommissionSchedule / CostEstimate | `trading/qmt_gateway_service.py`、`trading/oms.py` | fee fixture |
| FEAT12-TASK-06 | CR138-S06 | run / account summary | PnLSnapshot / ReturnSummary | `trading/qmt_gateway_service.py`、`trading/reconciliation.py` | PnL blocked / estimated fixture |
| FEAT12-TASK-07 | CR138-S07 | market auth / symbols | MarketSubscription state | `trading/qmt_gateway_service.py` | subscription fixture |
| FEAT12-TASK-08 | CR138-S07 | order intent / risk decision | GatewayCommand hard reject / future runtime boundary | `trading/qmt_gateway_gates.py`、`trading/oms.py` | adapter_calls=0 fixture |
| FEAT12-TASK-09 | CR138-S07 | execution event fixture | ExecutionReport / RecoveryPlan | `trading/qmt_gateway_service.py` | recovery fixture |
| FEAT12-TASK-10 | CR138-S08 | audit / config fixture | GatewayAuditRecord / GatewayChangePlan docs | docs / process checks | docs guardrail |

## 实施顺序

1. 先完成 CR138-S01 shared contract。
2. CR138-S05 冻结 REST-only lifecycle / health / capabilities。
3. CR138-S06 和 CR138-S07 可并行设计；实现时需按 shared gateway service merge owner 串行合并。
4. CR138-S08 汇总文档、fixtures 和 CP7 guardrails。

