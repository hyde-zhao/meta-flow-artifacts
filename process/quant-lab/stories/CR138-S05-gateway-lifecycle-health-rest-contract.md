---
story_id: "CR138-S05-gateway-lifecycle-health-rest-contract"
cr_id: "CR-138"
title: "Gateway lifecycle, health, capabilities, REST contract, and change plan"
status: "lld-ready-for-review"
priority: "P0"
wave: "CR138-W2-CONTROL-SHELL"
depends_on:
  - "CR138-S01-shared-contracts-authorization-audit"
dependency_type:
  - "contract"
feature_design_refs:
  - "process/docs/features/qmt-gateway-service-layer/DESIGN.md"
  - "process/docs/features/qmt-gateway-service-layer/TEST-PLAN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "external service boundary"
    - "REST-only P0"
    - "runtime authorization"
    - "gateway change plan"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S05 Gateway Lifecycle, Health, Capabilities, REST Contract, and Change Plan

## 用户价值

用户可以明确 Gateway 是否可用、具备哪些能力、当前 session 处于什么状态，并能对 Gateway 配置变更做 dry-run / rollback 检查，同时不会把 health pass 或 ChangePlan 误读成真实账户、运行或交易授权。

## 范围

- GatewayHealth、CapabilitySnapshot、TradingSession。
- REST-only P0 route contract。
- lifecycle / capabilities / session status 的 fixture 级验证。
- Gateway ChangePlan dry-run、compatibility check、rollback target required。

## 非范围

- 不启动真实 Gateway。
- 不绑定端口。
- 不连接 QMT / MiniQMT / XtQuant。
- 不实现 SSE / WebSocket / gRPC / FIX。
- 不执行真实 config apply / restart。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | REST-only P0 route registry 不包含 SSE / WebSocket / gRPC / FIX。 |
| AC-02 | GatewayHealth 支持 healthy / degraded / unavailable / disabled。 |
| AC-03 | CapabilitySnapshot 不升级 account / market / order / trading 授权。 |
| AC-04 | TradingSession account_label 必须脱敏。 |
| AC-05 | GatewayChangePlan 无 rollback target 时 rejected；dry-run 通过时 `apply_allowed=false`。 |

## 技术说明

需要 full LLD，重点说明 REST route、lifecycle 状态、capabilities scope、session redaction、Gateway ChangePlan dry-run 和 runtime-gated path。
