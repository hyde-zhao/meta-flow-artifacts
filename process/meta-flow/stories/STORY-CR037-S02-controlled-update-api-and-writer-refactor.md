---
story_id: "CR037-S02"
cr_ref: "CR-037"
title: "controlled update API and writer refactor"
status: "verified-with-risk"
wave: "CR037-W1"
priority: "P0"
depends_on: ["CR037-S01"]
feature_design_refs:
  - "process/docs/features/current-state-enforcement/DESIGN.md"
  - "process/docs/features/current-state-enforcement/TEST-PLAN.md"
  - "process/docs/features/current-state-enforcement/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module-contract", "rollback", "internal-writer"]
file_ownership:
  primary: ["meta_flow/state/current.py", "meta_flow/workflow/cr_lifecycle.py"]
  shared: ["tests/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S02 controlled update API and writer refactor

## 用户价值

作为维护者，我要所有 current state 写入走受控更新入口，以便写入前后都可校验。

## 验收标准

- 新增受控 `update_current_state()` 语义，patch key 也受 allowlist 校验。
- `cr_lifecycle` 中直接写 `STATE.current.json` 的路径改为受控 API。
- 写入失败不落盘，错误可被测试定位。

## 设计证据要求

必须生成 full LLD，明确 deep-merge、删除语义、内部调用点和回滚策略。
