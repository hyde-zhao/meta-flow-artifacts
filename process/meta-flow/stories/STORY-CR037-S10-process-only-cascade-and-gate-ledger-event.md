---
story_id: "CR037-S10"
cr_ref: "CR-037"
title: "process-only cascade and Gate Ledger event"
status: "dev-ready"
wave: "CR037-W4"
priority: "P1"
depends_on: ["CR037-S09"]
feature_design_refs:
  - "process/docs/features/roadmap-refresh-governance/DESIGN.md"
  - "process/docs/features/roadmap-refresh-governance/TEST-PLAN.md"
  - "process/docs/features/roadmap-refresh-governance/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["security", "runtime_authorization", "event-contract"]
file_ownership:
  primary: ["meta_flow/state/event_ledger.py", "meta_flow/project/**"]
  shared: ["process/state/GATE-LEDGER.ndjson", "tests/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S10 process-only cascade and Gate Ledger event

## 用户价值

作为审批者，我要 refresh cascade 只自动更新过程归档库，以避免误改发布库。

## 验收标准

- cascade 自动写入范围仅限 process artifact / project objects。
- 发布库陈旧项进入 stale / follow-up，不直接写。
- Gate Ledger 记录 `roadmap_refresh` 事件和 result ref。

## 设计证据要求

必须生成 full LLD，明确 target guard、ledger event 字段和 rollback。
