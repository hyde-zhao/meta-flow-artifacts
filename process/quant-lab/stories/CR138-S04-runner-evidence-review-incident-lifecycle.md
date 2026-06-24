---
story_id: "CR138-S04-runner-evidence-review-incident-lifecycle"
cr_id: "CR-138"
title: "Runner evidence, review, incident, and lifecycle change"
status: "lld-ready-for-review"
priority: "P1"
wave: "CR138-W4-REVIEW-DOCS-GUARDRAILS"
depends_on:
  - "CR138-S02-runner-plan-preflight-control"
  - "CR138-S03-runner-event-signal-rebalance-tracking"
  - "CR138-S06-gateway-query-calendar-commission-pnl"
  - "CR138-S07-gateway-subscription-order-report-recovery"
dependency_type:
  - "contract"
  - "runtime"
feature_design_refs:
  - "process/docs/features/runner-control-plane/DESIGN.md"
  - "process/docs/features/runtime-authorization-safety/TEST-PLAN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "evidence redaction"
    - "incident recovery"
    - "strategy lifecycle change"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S04 Runner Evidence, Review, Incident, and Lifecycle Change

## 用户价值

用户可以盘后复盘、定位异常、形成 follow-up，并安全地提出策略暂停 / 参数变更 / 版本回滚计划。

## 范围

- RunEvidence、ReviewSummary、IncidentRecord、RecoveryPlan 消费。
- StrategyChangePlan 和 rollback target 检查。
- raw sensitive data redaction。

## 非范围

- 不复制原始日志。
- 不读取真实账户 / 成交 / 委托。
- 不自动发布策略变更。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | RunEvidence 只输出 redacted summary 和 evidence refs。 |
| AC-02 | ReviewSummary 回链 run_id、audit_id、GatewayAuditRecord。 |
| AC-03 | unknown order state 进入 manual_takeover，不自动重发。 |
| AC-04 | StrategyChangePlan 缺 rollback_target 时 blocked。 |

## 技术说明

需要 full LLD，重点说明 evidence index、redaction、incident 路由、follow-up candidate 和策略变更不自动生效。

