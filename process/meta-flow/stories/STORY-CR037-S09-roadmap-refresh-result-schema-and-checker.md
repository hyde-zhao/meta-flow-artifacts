---
story_id: "CR037-S09"
cr_ref: "CR-037"
title: "roadmap refresh result schema and checker"
status: "dev-ready"
wave: "CR037-W4"
priority: "P1"
depends_on: ["CR037-S06", "CR037-S07"]
feature_design_refs:
  - "process/docs/features/roadmap-refresh-governance/DESIGN.md"
  - "process/docs/features/roadmap-refresh-governance/TEST-PLAN.md"
  - "process/docs/features/roadmap-refresh-governance/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "checker", "rollback"]
file_ownership:
  primary: ["meta_flow/checks/**", "schemas/**"]
  shared: ["process/checks/**", "tests/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S09 roadmap refresh result schema and checker

## 用户价值

作为维护者，我要 roadmap refresh 生成独立 result 和 checker，以区别于 CP gate。

## 验收标准

- `ROADMAP-REFRESH` 使用独立 result schema，不复用 CP result。
- checker 能校验 refresh 专属 decision、machine_updates、stale_items、follow_up_candidates。
- result 不允许声明发布库自动写入。

## 设计证据要求

必须生成 full LLD，明确 schema、checker、错误枚举和 CLI。
