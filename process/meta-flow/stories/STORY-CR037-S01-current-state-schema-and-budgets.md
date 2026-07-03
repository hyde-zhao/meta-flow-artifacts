---
story_id: "CR037-S01"
cr_ref: "CR-037"
title: "current-state schema and budgets"
status: "verified-with-risk"
wave: "CR037-W1"
priority: "P0"
depends_on: ["CR037-S00"]
feature_design_refs:
  - "process/docs/features/current-state-enforcement/DESIGN.md"
  - "process/docs/features/current-state-enforcement/TEST-PLAN.md"
  - "process/docs/features/current-state-enforcement/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["data-model", "security", "state-schema"]
file_ownership:
  primary: ["meta_flow/state/current.py", "tests/test_state_v2.py"]
  shared: ["meta_flow/cli.py"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S01 current-state schema and budgets

## 用户价值

作为维护者，我要用 allowlist 和字段预算约束 `STATE.current.json`，以防止轻量入口继续膨胀。

## 验收标准

- unknown field 在 audit 模式 WARN，在 enforce 模式 ERROR。
- `source_refs`、`open_risks`、`authz_policy_refs`、`next_action` 等字段有预算测试。
- `project_state_ref` 和 `pending_checklist_path` 等合法 optional keys 被显式放行。

## 设计证据要求

必须生成 full LLD，明确 allowlist 来源、预算表、错误模型、audit/enforce 切换和 fixture。
