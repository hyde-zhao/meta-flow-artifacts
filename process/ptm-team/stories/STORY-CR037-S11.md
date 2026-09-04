---
story_id: "CR037-S11"
story_slug: "docs"
cr_id: "CR-037"
title: "文档（SKILL/README/ptm-te 执行指导/limit 语法）"
priority: "P1"
wave: 4
status: "ready-for-verification"
depends_on: ["CR037-S10"]
dependency_type: "precedence"
assignee: null
dev_context: "产出 topo-planning README、ptm-te 执行指导更新、limit 语法文档（S2 limit-syntax.md 引用）、命名规则说明。文档内容由实现回填。"
validation_context: "文档与实现一致，limit 语法与命名说明可读。"
acceptance_criteria:
  - "topo-planning README 覆盖安装/使用/limit 语法"
  - "ptm-te 执行指导补充拓扑映射步骤说明"
  - "文档与实际实现一致"
output_files:
  - "skills/topo-planning/README.md"
  - "docs/ptm-te/执行指导.md"
file_ownership:
  primary:
    - "skills/topo-planning/README.md"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "waived"
  trigger_reasons:
    - "文档 Story，无需设计证据"
  rationale: "内容由实现回填"
  waiver_reason: "纯文档产出，无架构/契约设计点，waived 无需 LLD"
dev_gate: "cp5-approved"
---

# STORY-CR037-S11：文档

## 目标

产出 topo-planning README + ptm-te 执行指导更新 + limit 语法与命名文档。

## Waived 理由

纯文档产出（README/执行指导/说明），无架构或契约设计点，lld_policy=waived，由实现回填内容。

## 依赖

CR037-S10（测试完成验证后）。

## 设计证据

无（waived，waiver_reason 见 frontmatter）。
