---
story_id: "CR138-S01-shared-contracts-authorization-audit"
cr_id: "CR-138"
title: "Shared contracts, authorization, and audit boundary"
status: "lld-ready-for-review"
priority: "P0"
wave: "CR138-W1-SHARED-CONTRACTS"
depends_on: []
dependency_type: []
feature_design_refs:
  - "process/docs/features/runner-control-plane/DESIGN.md"
  - "process/docs/features/qmt-gateway-service-layer/DESIGN.md"
  - "process/docs/features/runtime-authorization-safety/DESIGN.md"
  - "process/docs/features/qmt-trading-governance/DESIGN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "shared contract"
    - "runtime authorization"
    - "audit id"
    - "cross-feature boundary"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S01 Shared Contracts, Authorization, and Audit Boundary

## 用户价值

作为后续 Runner / Gateway / OMS / Safety Story 的共同前置，本 Story 冻结命令、事件、授权和审计合同，避免各 Story 在 CP5 / CP6 阶段各自定义不兼容字段。

## 范围

- RunnerCommand、GatewayCommand、GatewayEvent、ExecutionReport、AuthorizationRecord、AuditRecord、request_id、audit_id、idempotency_key。
- fail-closed 授权语义和 no-real-operation counters。
- Runner 禁止直连 xtquant；Gateway health 不升级授权。

## 非范围

- 不实现真实 runtime。
- 不启动 Gateway、不连接 QMT、不读取凭据或账户。
- 不提交 / 撤销订单。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | shared contract 字段覆盖 command/event/report/auth/audit 5 类对象。 |
| AC-02 | 无授权时 runtime/account/market/order/submit/cancel 均 fail-closed。 |
| AC-03 | Runner 文件 forbidden import `xtquant` 计数为 0。 |
| AC-04 | 所有跨 Runner/Gateway 事件都有 request_id 或 audit_id。 |

## 技术说明

需要 full LLD，重点冻结 schema、状态、错误码、权限字段、文件 owner 和回退策略。

