---
story_id: "CR037-S06"
cr_ref: "CR-037"
title: "PROJECT-SCALE and roadmap objects"
status: "dev-ready"
wave: "CR037-W2"
priority: "P1"
depends_on: ["CR037-S05"]
feature_design_refs:
  - "process/docs/features/project-state-governance/DESIGN.md"
  - "process/docs/features/project-state-governance/TEST-PLAN.md"
  - "process/docs/features/project-state-governance/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "cross-module-contract"]
file_ownership:
  primary: ["process/project/**", "meta_flow/project/**"]
  shared: ["meta_flow/policies/**", "tests/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S06 PROJECT-SCALE and roadmap objects

## 用户价值

作为审批者，我要看到项目规模和 gate profile bias 的原因，以便理解后续门禁默认倾向。

## 验收标准

- `PROJECT-SCALE.yaml` 记录 `lite / standard / full` 三档、bias 和 reason。
- 不直接修改 `GATE-PROFILES.json`。
- `ROADMAP.yaml` / `MILESTONES.yaml` 作为项目级长期对象可被 checker 读取。

## 设计证据要求

必须生成 full LLD，明确 scale bias 与 gate profile 的边界。
