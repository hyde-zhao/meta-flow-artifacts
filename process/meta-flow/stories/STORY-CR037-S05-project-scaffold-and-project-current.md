---
story_id: "CR037-S05"
cr_ref: "CR-037"
title: "project scaffold and PROJECT.current"
status: "dev-ready"
wave: "CR037-W2"
priority: "P1"
depends_on: ["CR037-S01", "CR037-S02"]
feature_design_refs:
  - "process/docs/features/project-state-governance/DESIGN.md"
  - "process/docs/features/project-state-governance/TEST-PLAN.md"
  - "process/docs/features/project-state-governance/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "workspace-scaffold"]
file_ownership:
  primary: ["meta_flow/workspace/routing.py", "meta_flow/state/**"]
  shared: ["tests/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S05 project scaffold and PROJECT.current

## 用户价值

作为维护者，我要用 `PROJECT.current.json` 表达 refs-only 项目状态，以承载长期项目治理。

## 验收标准

- bootstrap / workspace check 覆盖 `process/project/`。
- `PROJECT.current.json` 有 allowlist 和 size budget。
- `STATE.current.json` 只通过 `project_state_ref` 指向项目状态。

## 设计证据要求

必须生成 full LLD，明确 schema、budget、scaffold、读写路径和兼容迁移。
