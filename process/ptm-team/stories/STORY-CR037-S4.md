---
story_id: "CR037-S4"
story_slug: "topo-mapper-enhance"
cr_id: "CR-037"
title: "topo_mapper.py 增强（per-link port_requirement + hardware_platform + ledger 占用闭环 + mapping_validator 兼容）"
priority: "P0"
wave: 2
status: "lld-ready-for-review"
depends_on: ["CR037-S1", "CR037-S2", "CR037-S3"]
dependency_type: "precedence"
assignee: null
dev_context: "增强 skills/topo-config/src/topo_mapper.py：match(topo, port_requirement) 支持 per-link 约束（None/全局 dict/per-link dict 三态）；回溯在 _check_connectivity/_port_meets_requirement 按 link_id 剪枝；hardware_platform 等价类匹配（对齐 S2/S3）；端口兼容矩阵内联常量；H1 ledger 占用闭环（_is_port_busy 消费 allocated_to + 映射成功写回 allocate + 新增 PhysicalPool.release(env_name)）；H4 约束叠加（用户 > 逻辑 topo > 默认；**S4-A：用户 speed_class 为 TE 系时 media_type 强制 fiber、丢弃逻辑 copper，不产出 {TE, copper}**）；**S4-B：include-at-least-one 软约束分支（not strict）必须位于硬剪枝 return False 之前**；P1-6 mapping_validator.py 纳入范围，R2 按 PortMapping.via_link per-link 聚合不静默跳过。**S3-B：TopoMapper 输入 raw PhysicalPool（由 S3 最终池数据经 PhysicalPool.from_pool_data() 构造，physical_pool.py 新增 from_pool_data 加性构造器）**。"
validation_context: "物理池含 TE 口时 per-link TE 约束映射成功；不含时返回结构化失败报告（缺 TE 端口）+ --report-out 落盘。ledger 占用闭环：映射成功后 allocated_to 写回，二次映射避开已占用端口，release(env_name) 释放后恢复 free。约束叠加优先级单测覆盖。mapping_validator R2 校验不因 per-link 改造静默跳过。"
acceptance_criteria:
  - "match() 支持 port_requirement 三态（None / 全局 dict / per-link {link_id: {speed_class, media_type}}）"
  - "回溯剪枝：per-link 约束在 _check_connectivity/_port_meets_requirement 按 link_id 应用"
  - "hardware_platform 等价类匹配（HG3250/JL-HG 等）与 device_type 并列候选过滤"
  - "端口兼容矩阵：GE 满足 copper|fiber，TE/XTE/QTE 仅 fiber；无 TE 口时 TE 约束判 failed"
  - "H1 ledger 闭环：_is_port_busy 消费台账 allocated_to；映射成功写回 allocate+save_ledger；PhysicalPool.release(env_name) 释放（台账 schema 不变 P0-3，S4-C）"
  - "P1-7 释放归属：S4 定义 release 接口，S8 挂接 ptm-te 编排 [7] teardown"
  - "H4 叠加（S4-A）：用户 topo-limit > 逻辑 topo 自带（link.media_type + 端点接口 speed_class）> 默认；用户 speed_class 为 TE 系时 media_type 强制 fiber、丢弃逻辑 copper，不产出 {TE, copper}"
  - "P1-6（S4-D）：mapping_validator.py R2 链路类型校验 per-link 聚合，无约束端口计入 checked 标注 unconstrained，不静默跳过"
  - "include-at-least-one 软约束（S4-B）：not strict 分支位于硬剪枝 return False 之前 + post-match 计数 ≥1，0 条满足则失败并产出结构化明细"
output_files:
  - "skills/topo-config/src/topo_mapper.py"
  - "skills/topo-config/src/topology_model.py"
  - "skills/topo-config/src/mapping_validator.py"
  - "skills/topo-config/src/physical_pool.py#release + #from_pool_data（仅新增方法，不改台账 schema，S3-B/P1-7）"
file_ownership:
  primary:
    - "skills/topo-config/src/topo_mapper.py"
  shared:
    - "skills/topo-config/src/mapping_validator.py"
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "回溯匹配剪枝接入，改动核心引擎"
    - "ledger 占用闭环（H1）跨用例防冲突"
    - "mapping_validator 兼容（P1-6）"
  rationale: "per-link 约束、占用写回/释放、约束叠加优先级、R2 兼容需完整设计"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S4-LLD.md"
---

# STORY-CR037-S4：topo_mapper.py 增强

## 目标

增强 `topo_mapper.py`：per-link 端口约束（回溯剪枝）、`hardware_platform` 等价类匹配、H1 ledger 占用闭环（写回+释放）、H4 约束叠加、P1-6 mapping_validator 兼容。

## 依赖

CR037-S1（topology_collection）、CR037-S2（limit_parser）、CR037-S3（pool_merge）。

## 输出文件

- `skills/topo-config/src/topo_mapper.py`
- `skills/topo-config/src/topology_model.py`
- `skills/topo-config/src/mapping_validator.py`（P1-6）
- `skills/topo-config/src/physical_pool.py#release`（P1-7，仅新增方法）

## AI 任务清单

- [ ] TASK-S4-01：`match()` 支持 port_requirement 三态（None/全局/per-link）
- [ ] TASK-S4-02：回溯剪枝按 link_id 取约束
- [ ] TASK-S4-03：`hardware_platform` 等价类 + device_types 候选过滤
- [ ] TASK-S4-04：端口兼容矩阵常量 + `_port_meets_requirement` 按矩阵判定
- [ ] TASK-S4-05：H1 ledger 闭环（_is_port_busy 消费 allocated_to + allocate 写回 + save_ledger）
- [ ] TASK-S4-06：`PhysicalPool.release(env_name)` 释放接口（P1-7，台账 schema 不变）
- [ ] TASK-S4-07：H4 约束叠加（用户 > 逻辑 topo > 默认，并集语义）
- [ ] TASK-S4-08：`mapping_validator.py` R2 per-link 聚合（P1-6，不静默跳过）
- [ ] TASK-S4-09：include-at-least-one 软约束 post-match 计数

## 设计证据

见 `process/stories/STORY-CR037-S4-LLD.md`（14 章节 full-lld）。
