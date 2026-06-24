---
feature_id: "FEAT-11"
feature_name: "Runner Control Plane"
status: "ready-for-cp5-review"
version: "1.0"
change: "CR-138"
created_at: "2026-06-24T16:10:00+08:00"
created_by: "host-orchestrator"
---

# FEAT-11 Runner Control Plane Tasks

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-24 | host-orchestrator | 初版任务清单，供 CR138 Story LLD 拆分和 CP5 批量确认 |

## 任务清单

| TASK ID | Story | 输入 | 输出 | 文件范围 | 验证入口 |
|---|---|---|---|---|---|
| FEAT11-TASK-01 | CR138-S01 | HLD / ADR / Domain Map | shared command / event / audit contract 冻结 | `trading/runner_control_contracts.py`、`trading/qmt_gateway_contracts.py` | contract fixture |
| FEAT11-TASK-02 | CR138-S02 | FEAT11-TASK-01 | RunPlan / PreflightResult | `trading/runner_control_plane.py` | `tests/test_cr138_runner_control_plane.py` |
| FEAT11-TASK-03 | CR138-S02 | RunPlan / GatewayHealth / AuthorizationRecord | preflight gate fail-closed | `trading/runner_control_plane.py` | authorization fixture |
| FEAT11-TASK-04 | CR138-S03 | signal / event fixture | idempotent event ingestion | `trading/runner_control_plane.py` | duplicate event fixture |
| FEAT11-TASK-05 | CR138-S03 | OrderIntentDraft / risk fixture | rebalance intent + risk blocked path | `trading/runner_control_plane.py`、`trading/oms.py` | no adapter call fixture |
| FEAT11-TASK-06 | CR138-S04 | GatewayEvent / ExecutionReport | RunState / IncidentRecord | `trading/runner_control_plane.py` | recovery scenario fixture |
| FEAT11-TASK-07 | CR138-S04 | evidence refs | redacted RunEvidence / ReviewSummary | `trading/runner_control_plane.py`、`trading/strategy_runner/evidence_index.py` | redaction fixture |
| FEAT11-TASK-08 | CR138-S04 | version / parameter diff | StrategyChangePlan | `trading/runner_control_plane.py` | rollback target fixture |
| FEAT11-TASK-09 | CR138-S08 | CP7 / docs boundary | runbook 不授权声明 | docs / process checks | docs guardrail |

## 实施顺序

1. 先完成 CR138-S01 合同和安全边界。
2. 并行设计 CR138-S02 与 CR138-S05；实现时依赖 S01 合同冻结。
3. 在 S02/S05 后推进 S03/S06/S07。
4. 最后完成 S04/S08 的 evidence、review、runbook 和 CP7 guardrail。

