---
story_id: "CR037-S08"
cr_ref: "CR-037"
title: "impact surface field split and migration report"
status: "dev-ready"
wave: "CR037-W3"
priority: "P1"
depends_on: ["CR037-S07"]
feature_design_refs:
  - "process/docs/features/impact-surface-normalization/DESIGN.md"
  - "process/docs/features/impact-surface-normalization/TEST-PLAN.md"
  - "process/docs/features/impact-surface-normalization/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["migration", "data-model", "compatibility"]
file_ownership:
  primary: ["meta_flow/workflow/cr_lifecycle.py", "meta_flow/checks/cr_tracking.py"]
  shared: ["tests/**", "process/changes/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S08 impact surface field split and migration report

## 用户价值

作为维护者，我要把 impact surface、路径、feature、capability 分开，以恢复冲突检测语义。

## 验收标准

- 新 CR 使用治理面枚举，不把文件路径写进 impact surface。
- 历史漂移只生成 migration report，不刷屏阻断旧 CR。
- `affected_paths`、`feature_refs`、`capability_refs` 各自有校验语义。

## 设计证据要求

必须生成 full LLD，明确 enum、migration scanner、audit/enforce 和 resolver 依赖。
