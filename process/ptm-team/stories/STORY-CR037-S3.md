---
story_id: "CR037-S3"
story_slug: "pool-merge"
cr_id: "CR-037"
title: "pool_merge.py（devices.yaml × physical_pool.yaml 归并：firewall host 关联 + TG api_server 关联 + 台账三角映射）"
priority: "P0"
wave: 1
status: "lld-ready-for-review"
depends_on:
  - "P-1"
dependency_type: "precedence"
assignee: null
dev_context: "新建 skills/topo-planning/src/pool_merge.py，归并目标项目 topology/physical_pool.yaml + topology/devices.yaml 为最终池数据（UnifiedPool）。核心关联：① device_groups[].firewall.host ↔ pool.nodes[].management.host（DUT 节点关联；**device_groups 实为 list**，S3-A，遍历用 list 迭代）；② device_groups[].tg.api_server ↔ pool.nodes[].management.api_server（TG 节点关联，H2，关联前归一化 api_server：去 scheme / 统一 IP:port，P1-4；**api_server 主键 + host 兜底**，S3-C）；③ 台账三角映射（台账.model ↔ 池.device_type ↔ device-reference 平台，台账为 list + port_status schema，按 device_id 关联，P0-3；model 与 device_type 可能同值直通，P2-5；**型号冲突消歧统一在 S2 平台别名层，S3 不重复决策**，S3-D）。冲突处理：同 host 型号不同以 devices.yaml 为准 + warning；池全量承载时降级为仅校验 devices 存在性（备选 B）。归并节点 node_id 取 pool_node.get('node_id', node_key)（P1-5）。**S3-B（评审 D 组契约）**：UnifiedPool 是最终池数据，不是 S4 topo_mapper 的直接输入；S4/S7 用 `UnifiedPool.to_pool_data()` + `ledger_path` 经 `PhysicalPool.from_pool_data()` 构造 raw PhysicalPool。"
validation_context: "归并池包含每个节点的完整视图（接线+型号+平台+管理地址）。host 关联匹配率 100%（存在对应 host 的前提）。TG api_server 关联成功。台账三角映射可查。冲突场景（同 host 不同 device_type）有 warning 且以 devices.yaml 为准。池全量承载降级路径可用。"
acceptance_criteria:
  - "merge_pool(pool_path, devices_path, ledger_path) → UnifiedPool 对象（最终池数据，S3-B：S4/S7 用 to_pool_data() + ledger_path 构造 raw PhysicalPool）"
  - "DUT 节点：pool.nodes[].management.host ↔ devices.device_groups[].firewall.host 关联；device_groups 为 list，遍历用 list 迭代（S3-A），不得 .items()/.values()"
  - "TG 节点：pool.nodes[].management.api_server ↔ devices.device_groups[].tg.api_server 关联（归一化后匹配，P1-4；api_server 主键 + host 兜底，S3-C）"
  - "关联失败（host/api_server 无匹配）时不阻断，标记 unmatched_nodes[]"
  - "台账三角映射：ledger.model → pool.device_type → device-reference 平台（台账为 list + port_status schema，按 device_id 关联，P0-3）可查询；型号冲突消歧统一在 S2，S3 不重复决策（S3-D）"
  - "型号冲突：同 host 且 pool.device_type ≠ devices.device_group.device_type → warning + 以 devices 为准"
  - "池全量承载降级：若物理池节点无需 devices 补充 → 仅校验 devices 存在性"
  - "TG 节点归并：从 devices 补充 sub_type（ixia-c/trex）+ hardware_platform"
output_files:
  - "skills/topo-planning/src/pool_merge.py"
file_ownership:
  primary:
    - "skills/topo-planning/src/pool_merge.py"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "多数据源归并 + 冲突处理需完整设计"
    - "TG 归并规则（H2）"
  rationale: "host 关联规则、TG api_server 关联、冲突消歧、三角映射需明确契约"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S3-LLD.md"
---

# STORY-CR037-S3：pool_merge.py

## 目标

新建 `skills/topo-planning/src/pool_merge.py`，归并目标项目 `topology/physical_pool.yaml` 与 `topology/devices.yaml` 为统一物理池视图（UnifiedPool），完成 firewall host 关联、TG api_server 关联、台账三角映射三条归并路径。

## 依赖

P-1（物理池模板化）——内嵌的 `physical_pool.yaml` schema 形态决定解析方式。

## 输出文件

`skills/topo-planning/src/pool_merge.py`

## AI 任务清单

- [ ] TASK-S3-01：定义 `UnifiedPool` / `UnifiedNode` 数据类（含 `to_pool_data()` / `to_pool_node()`，S3-B）
- [ ] TASK-S3-02：实现 `load_pool(pool_path) → PoolData` 加载物理池
- [ ] TASK-S3-03：实现 `load_devices(devices_path) → DevicesData` 加载设备清单（`device_groups` 为 list，S3-A）
- [ ] TASK-S3-04：实现 `load_ledger(ledger_path) → LedgerData` 加载台账
- [ ] TASK-S3-05：实现 DUT host 关联（`device_groups[].firewall.host ↔ pool.nodes[].management.host`，list 遍历，S3-A）
- [ ] TASK-S3-06：实现 TG api_server 关联（`device_groups[].tg.api_server ↔ pool.nodes[].management.api_server`，H2；api_server 主键 + host 兜底，S3-C；**显式决策表：先完整遍历全部 candidates 找 api_server 主键，再完整遍历找 host 兜底，禁止单候选内 api_server+host 同判导致首组误命中，Minor-1**）
- [ ] TASK-S3-07：实现 台账三角映射（`ledger.model ↔ pool.device_type ↔ device-reference 平台`；消歧统一在 S2，S3 不重复决策，S3-D）
- [ ] TASK-S3-08：实现 冲突处理（型号不一致→warning + devices 为准）
- [ ] TASK-S3-09：实现 池全量承载降级（仅校验 devices 存在性）
- [ ] TASK-S3-10：输出 unmatched_nodes[] 清单（关联失败节点）

## 设计证据

见 `process/stories/STORY-CR037-S3-LLD.md`（14 章节 full-lld）。
