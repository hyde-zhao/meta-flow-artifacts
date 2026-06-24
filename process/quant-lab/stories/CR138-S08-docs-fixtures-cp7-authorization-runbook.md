---
story_id: "CR138-S08-docs-fixtures-cp7-authorization-runbook"
cr_id: "CR-138"
title: "Docs, fixtures, CP7 guardrails, and authorization runbook"
status: "lld-ready-for-review"
priority: "P1"
wave: "CR138-W4-REVIEW-DOCS-GUARDRAILS"
depends_on:
  - "CR138-S01-shared-contracts-authorization-audit"
  - "CR138-S02-runner-plan-preflight-control"
  - "CR138-S03-runner-event-signal-rebalance-tracking"
  - "CR138-S04-runner-evidence-review-incident-lifecycle"
  - "CR138-S05-gateway-lifecycle-health-rest-contract"
  - "CR138-S06-gateway-query-calendar-commission-pnl"
  - "CR138-S07-gateway-subscription-order-report-recovery"
dependency_type:
  - "contract"
  - "file-conflict"
feature_design_refs:
  - "process/docs/features/runner-control-plane/TASKS.md"
  - "process/docs/features/qmt-gateway-service-layer/TASKS.md"
  - "process/docs/features/runtime-authorization-safety/TEST-PLAN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "authorization runbook"
    - "CP7 guardrail"
    - "docs safety statement"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S08 Docs, Fixtures, CP7 Guardrails, and Authorization Runbook

## 用户价值

用户可以在 CP7 / CP8 前清楚知道 CR138 交付到了哪一层、哪些是真实授权、哪些只是 fixture / mock / static 证明，避免把设计或健康检查误读为可交易能力。

## 范围

- README / USER-MANUAL / runbook 增量候选。
- CR138 fixture matrix 和 CP7 guardrail。
- runtime_authorization gate 的后续模板说明。

## 非范围

- 不真实运行。
- 不读凭据或账户。
- 不执行 Git remote 写入。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | 文档列出不授权项和按需授权路径。 |
| AC-02 | fixture 覆盖 Runner/Gateway happy / failure path。 |
| AC-03 | no-real-operation counters 对所有禁止动作为 0。 |
| AC-04 | CP7 验证计划区分 static / fixture / mock / runtime_authorized。 |

## 技术说明

需要 full LLD，因为本 Story 直接承载授权语义、验证声明和发布前用户可见 runbook，不能用轻量 technical-note 隐藏安全细节。

