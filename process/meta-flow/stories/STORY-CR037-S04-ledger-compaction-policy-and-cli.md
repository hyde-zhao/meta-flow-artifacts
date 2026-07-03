---
story_id: "CR037-S04"
cr_ref: "CR-037"
title: "ledger compaction policy and CLI"
status: "verified-with-risk"
wave: "CR037-W1A"
priority: "P0.5"
depends_on: ["CR037-S01"]
feature_design_refs:
  - "process/docs/features/ledger-compaction/DESIGN.md"
  - "process/docs/features/ledger-compaction/TEST-PLAN.md"
  - "process/docs/features/ledger-compaction/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["migration", "rollback", "audit"]
file_ownership:
  primary: ["meta_flow/state/event_ledger.py", "meta_flow/cli.py"]
  shared: ["tests/**", "process/policies/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S04 ledger compaction policy and CLI

## 用户价值

作为维护者，我要 ledger compaction 与 state compact 在命令和职责上明确分离，以避免长期审计数据治理和 current state 渲染语义混淆。

## 验收标准

- 使用 `meta-flow ledger compact` 或 CP5 确认后的等价命令，不复用 `state compact`。
- 默认 dry-run，apply 需要显式授权。
- archive index、hash 和 rollback 策略可审计。

## 设计证据要求

必须生成 full LLD，明确 retention policy、archive marker、event checker compatibility 和恢复路径。
