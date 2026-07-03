---
story_id: "CR037-S12"
cr_ref: "CR-037"
title: "project stale-check"
status: "dev-ready"
wave: "CR037-W4"
priority: "P1"
depends_on: ["CR037-S09", "CR037-S11"]
feature_design_refs:
  - "process/docs/features/project-stale-check/DESIGN.md"
  - "process/docs/features/project-stale-check/TEST-PLAN.md"
  - "process/docs/features/project-stale-check/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-object-semantics", "follow_up_tracking"]
file_ownership:
  primary: ["meta_flow/checks/**", "meta_flow/cli.py"]
  shared: ["tests/**", "process/checks/**"]
  forbidden: ["process/quant-lab/**"]
---

# CR037-S12 project stale-check

## 用户价值

作为维护者，我要 project stale-check 输出跨对象语义陈旧 finding，以持续追踪 roadmap 影响。

## 验收标准

- stale-check 专注语义陈旧，不重复 cr-tracking 结构检查。
- finding 可转 FU-RF / SP-RF / RA-RF。
- checker 不自动修改 HLD、TEST-STRATEGY、release docs 或发布库。

## 设计证据要求

必须生成 full LLD，明确 rule engine、finding schema、no-auto-fix guard 和 false positive 控制。
