---
story_id: "CR037-S7"
story_slug: "topo-planning-skill-cli"
cr_id: "CR-037"
title: "topo-planning skill + CLI topo-plan（向导 5 步 + 结构化入参）"
priority: "P0"
wave: 3
status: "lld-ready"
depends_on: ["CR037-S1", "CR037-S2", "CR037-S3", "CR037-S5", "CR037-S6"]
dependency_type: "precedence"
assignee: null
dev_context: "新建 skills/topo-planning/ skill：向导 5 步（展示池→选/建逻辑 topo→映射→导出→部署/验证）+ CLI topo-plan（--topo-id/--case-file/--limit/--limit-file/--devices-yaml/--pool-path/--ledger-path/--output/--report-out/--dry-run/--execute/--authorized）。NLP→CLI 由 agent 依据 S2 limit-syntax.md 转换（G5/DQ-037-06）；统一 PYTHONPATH import 集成（S1 P2-4）；默认 dry-run 门（不写台账/不落真实路径，--output 指向 run 目录时落预览副本 executable:false；--execute 必须配 --authorized 显式门写台账+落盘，未授权抛 RuntimeAuthorizationError；真机部署由 S9 承接，DQ-037-04 独立授权）。"
validation_context: "CLI dry-run 不写台账/不落真实路径（0 次真机写）；--execute + --authorized 写台账占用 + 落盘 topology/topology.yaml；--execute 无 --authorized 抛 RuntimeAuthorizationError；--case-file 双源提取 topo_id（组网约束优先/组网描述回退）。向导 5 步链路 dry-run 验证通过。"
acceptance_criteria:
  - "CLI topo-plan 支持 12 参数（含 --output/--report-out/--dry-run/--execute/--authorized）"
  - "NLP→CLI 由 agent 依据 limit-syntax.md 转换（S2 交付物）"
  - "统一 PYTHONPATH import 集成（S1 P2-4），无 importlib 临时代码"
  - "dry-run 默认门：不写台账/不落真实路径；--output 指向 run 目录时落预览副本 executable:false；--execute 必须配 --authorized 显式门（未授权抛 RuntimeAuthorizationError）"
  - "--case-file 双源提取 topo_id（E2）"
  - "向导 5 步链路 dry-run 通过"
output_files:
  - "skills/topo-planning/SKILL.md"
  - "skills/topo-planning/scripts/topo_plan_cli.py"
  - "skills/topo-planning/templates/physical_pool.yaml.example"
  - "skills/topo-planning/templates/physical_ledger.yaml.example"
  - "skills/topo-planning/docs/limit-syntax.md"
file_ownership:
  primary:
    - "skills/topo-planning/SKILL.md"
    - "skills/topo-planning/scripts/topo_plan_cli.py"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "新 skill + CLI，Prompt-Skill 边界需完整设计"
  rationale: "skill 编排 + CLI 契约需完整设计"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S7-LLD.md"
---

# STORY-CR037-S7：topo-planning skill + CLI

## 目标

新建 `topo-planning` skill：向导 5 步 + CLI `topo-plan`，编排 topo 映射全流程。

## 依赖

CR037-S1/S2/S3/S5/S6。

## 输出文件

见 frontmatter `output_files`。

## AI 任务清单

- [ ] TASK-S7-01：CLI topo-plan 12 参数解析 + dry-run 默认门 + `--execute` 需 `--authorized` 显式门（RuntimeAuthorizationError）
- [ ] TASK-S7-02：向导 5 步流程（展示池→选/建→映射→导出→部署/验证）
- [ ] TASK-S7-03：NLP→CLI agent 转换指引（引用 S2 limit-syntax.md）
- [ ] TASK-S7-04：PYTHONPATH import 集成（S1 P2-4）
- [ ] TASK-S7-05：SKILL.md 编写

## 设计证据

见 `process/stories/STORY-CR037-S7-LLD.md`（full-lld）。
