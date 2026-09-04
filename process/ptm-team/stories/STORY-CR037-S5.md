---
story_id: "CR037-S5"
story_slug: "exporter"
cr_id: "CR-037"
title: "exporter.py（映射结果 → env-file + 命名规则 + link 数一致性 + 凭据禁止 + 结构化失败报告）"
priority: "P0"
wave: 2
status: "lld-ready-for-review"
depends_on: ["CR037-S4"]
dependency_type: "precedence"
assignee: null
dev_context: "新建 skills/topo-planning/src/exporter.py：映射结果（设备/端口/link + IP 规划）→ env-file（metadata+port_mapping+nodes+links，9 类 ${ENV.*} 占位符硬契约，端口名引号化防 YAML 1.1 整数陷阱；metadata 含 executable/degraded_reason）。命名默认 topology/topology.yaml + --output 描述性命名。link 数一致性（真实逻辑 link = 不含 Mock 端点、不含 L2 透传（auto_inserted_switch）的逻辑 link 数 vs 实际直连 matched 数，不一致报 LINK_COUNT_MISMATCH）。凭据禁止（ADR-02，正则断言无 password/token/api_key/secret）。结构化失败报告 JSON+人读双格式 + --report-out 落盘（O-c），映射失败不产出部分 env-file。IP 规划接入（权威 schema=ipam.ip_plan_to_dict）：缺 IP 默认抛 IP_PLAN_REQUIRED/IP_PLAN_INCOMPLETE，不允许静默空 IP（resolve_env_refs 对空 IP 抛 ValueError，op_mapper.py:1123-1125）；仅显式降级（dry-run/--allow-degraded）产出 executable=false 降级产物且不进入 case-execution 消费路径。MVP 边界（R1）：仅支持单 TG + 单 DUT 直连（node2_dut1_tg1_linkN 系列），_validate_mvp_topology 校验节点 node_type 分布，多节点（≥2 DUT）/含 SW/Mock/PC 拓扑抛 TOPO_NODE_COUNT_UNSUPPORTED + 结构化明细，不产出缺 dutN 坏产物；HLD §1.3 非目标补充该边界。确定性（R2）：generated_at 移出确定性比较范围（剔除后比较）或 _build_metadata 注入固定值。"
validation_context: "产物 env-file 通过 case-execution load_env_file 校验（9 类占位符 + 端口名引号）。link 数一致性错误路径有单测（LINK_COUNT_MISMATCH）。产物正则断言无凭据。失败报告 --report-out 落盘 JSON。多节点/SW/Mock 拓扑拒绝有单测（TOPO_NODE_COUNT_UNSUPPORTED，R1）。确定性单测剔除/注入 generated_at 后逐字节一致（R2）。"
acceptance_criteria:
  - "映射结果 → env-file 四段（metadata+port_mapping+nodes+links），满足 9 类 ${ENV.*} 占位符硬契约；metadata 含 executable/degraded_reason"
  - "端口名加引号（YAML 1.1 整数陷阱 Gotcha #11）"
  - "命名默认 topology/topology.yaml；--output 支持描述性命名"
  - "link 数一致性：真实逻辑 link（不含 Mock 端点、不含 L2 透传）= 实际直连 matched 数，不一致报 LINK_COUNT_MISMATCH"
  - "凭据禁止：产物正则断言无 password/token/api_key/secret（ADR-02）"
  - "结构化失败报告 JSON+人读，缺设备/缺TE端口/链路约束不满足明细，--report-out 落盘（O-c）"
  - "映射失败不产出部分 env-file"
  - "IP 规划接入（权威 schema=ipam.ip_plan_to_dict）：缺 IP 默认报错（IP_PLAN_REQUIRED/IP_PLAN_INCOMPLETE），不允许静默空 IP；仅显式降级产出 executable=false 降级产物且不进入 case-execution 消费路径"
  - "MVP 边界（R1）：仅支持单 TG + 单 DUT 直连；多节点（≥2 DUT）/含 SW/Mock/PC 拓扑显式抛 TOPO_NODE_COUNT_UNSUPPORTED + 结构化节点分布明细，不产出缺 dutN 坏产物"
  - "确定性（R2）：generated_at 移出确定性比较范围（剔除后比较）或经 _build_metadata 注入固定值；同输入同输出逐字节一致"
output_files:
  - "skills/topo-planning/src/exporter.py"
file_ownership:
  primary:
    - "skills/topo-planning/src/exporter.py"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "产物契约（9 类占位符）与命名/报告需完整设计"
  rationale: "env-file 契约 + 失败报告 schema 需明确"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S5-LLD.md"
---

# STORY-CR037-S5：exporter.py

## 目标

新建 `exporter.py`：映射结果 → ptm-te env-file，含命名规则、link 数一致性校验、凭据禁止、结构化失败报告。

## 依赖

CR037-S4（topo_mapper 增强）。

## 输出文件

`skills/topo-planning/src/exporter.py`

## AI 任务清单

- [ ] TASK-S5-01：env-file 生成（metadata+port_mapping+nodes+links，9 类占位符）
- [ ] TASK-S5-02：端口名引号化（Gotcha #11）
- [ ] TASK-S5-03：命名规则（默认 topology.yaml + --output 描述性）
- [ ] TASK-S5-04：link 数一致性校验（LINK_COUNT_MISMATCH）
- [ ] TASK-S5-05：凭据禁止正则断言（ADR-02）
- [ ] TASK-S5-06：结构化失败报告 JSON+人读 + --report-out 落盘（O-c）
- [ ] TASK-S5-07：映射失败不产出部分 env-file
- [ ] TASK-S5-08：MVP 边界校验 `_validate_mvp_topology`（R1：多节点/含 SW/Mock 拓扑 → TOPO_NODE_COUNT_UNSUPPORTED + 分布明细）
- [ ] TASK-S5-09：确定性（R2：generated_at 移出比较范围 / `_build_metadata` 支持注入固定值）

## 设计证据

见 `process/stories/STORY-CR037-S5-LLD.md`（14 章节 full-lld）。
