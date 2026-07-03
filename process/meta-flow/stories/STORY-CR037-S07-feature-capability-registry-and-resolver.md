---
story_id: "CR037-S07"
cr_ref: "CR-037"
title: "feature/capability registry and resolver"
status: "dev-ready"
wave: "CR037-W3"
priority: "P1"
depends_on: ["CR037-S05"]
feature_design_refs:
  - "process/docs/features/capability-feature-registry/DESIGN.md"
  - "process/docs/features/capability-feature-registry/TEST-PLAN.md"
  - "process/docs/features/capability-feature-registry/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "external-interface"]
file_ownership:
  primary: ["meta_flow/design/**", "docs/design/FEATURE-REGISTRY.yaml"]
  shared: ["tests/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S07 feature/capability registry and resolver

## 用户价值

作为迁移执行者，我要 capability / feature refs 只引用标准 registry，以避免能力名称漂移。

## 验收标准

- 未注册 ID 进入 blocked finding 或 candidate report，不自动创造 canonical ID。
- resolver 能区分 resolved、unresolved、deprecated、conflict。
- 下游 impact normalization、roadmap refresh、migration 只消费 registry-backed refs。

## 设计证据要求

必须生成 full LLD，明确 registry schema、resolver API 和 candidate report。
