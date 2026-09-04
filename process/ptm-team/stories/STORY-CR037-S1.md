---
story_id: "CR037-S1"
story_slug: "topology-collection"
cr_id: "CR-037"
title: "topology_collection.py（collection.md YAML 提取 + find_topology_file + index.yaml + min_ports 推导）"
priority: "P0"
wave: 1
status: "lld-ready-for-review"
depends_on: []
dependency_type: "none"
assignee: null
dev_context: "新建 skills/topo-planning/src/topology_collection.py，作为 topo-planning skill 的第一个模块。解析 resource/network-topology/topology-collection.md 提取 YAML 代码块，提供 find_topology_file(topo_id) 查找函数，支持 index.yaml 索引与 compatible_consumers 校验。补齐 topo-config/parser.py 缺失依赖 `from topology_collection import find_topology_file`。从 interfaces 推导 min_ports（collection 无该字段，接口数=端口需求）。"
validation_context: "12 个标准拓扑 find_topology_file 命中率 100%（返回 YAML 文件路径，供 parser.py detect_topology_format + _parse_yaml_topology 消费）。node2_dut1_tg1_link3 双源提取（组网约束/组网描述）均通过。index.yaml 校验 tgfw-topo-collection 条目 compatible_consumers 含 ptm-te，且 topo_id 在 collection.md 解析出的 key 集合中（不要求 node* 顶层条目）。list_topologies 候选集 = _parse_collection_md 解析出的 topo_id key 集合，返回 12 个 node* 拓扑（S1-A）。模糊搜索候选集同步为 _parse_collection_md 解析结果，编辑距离 ≤2 返回真实候选（node2_dut1_tg1_link3/link2/link5），不含不存在的 node4_dut1_tg1_link4（R3）。YAML 块字段分布以 collection.md 实况为准：仅 node2_dut1_tg1_link2 含 speed_class/media_type 全字段，其余 11 个为简写 {link: linkN}，缺省字段由推导/下游补充（S1-B）。YAML 提取含 metadata/nodes/links 完整结构。"
acceptance_criteria:
  - "find_topology_file(topo_id) 可查找全部 12 个拓扑，返回 YAML 文件路径（str，写入标准缓存路径，P0-1/P1-1）"
  - "解析 collection.md 提取 `## {topo_id}` 后第一个 YAML 代码块为合法 YAML"
  - "读 index.yaml 校验 tgfw-topo-collection 条目 compatible_consumers 含 ptm-te；topo_id 存在性以 collection.md 解析出的 key 集合为准（P0-2）"
  - "list_topologies() 候选集以 _parse_collection_md 解析出的 topo_id key 集合为准，返回 12 个 node* 拓扑；index.yaml 顶层仅 2 个 collection 级资源条目，不参与候选集（S1-A）"
  - "LLD §5 输入示例与 topology-collection.md 真实字段分布一致：仅 node2_dut1_tg1_link2 含 speed_class/media_type 全字段，其余 11 个为简写 {link: linkN}，缺省字段由推导/下游补充，不虚造（S1-B）"
  - "从 nodes[].interfaces 推导 min_ports：每节点接口数 = min_ports（无该字段时自动推导）"
  - "parser.py 可正常 `from topology_collection import find_topology_file`（安装后路径可及，统一 PYTHONPATH）"
  - "collection.md 不存在时返回清晰错误（FileNotFoundError + 提示检查 resource 安装）"
  - "topo_id 不存在于 collection key 集合时返回清晰错误（FileNotFoundError + 列出可用 topo_id）"
  - "YAML 解析失败时返回清晰错误（含文件名 + 行号）"
output_files:
  - "skills/topo-planning/src/topology_collection.py"
file_ownership:
  primary:
    - "skills/topo-planning/src/topology_collection.py"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "新建解析模块，跨资源格式"
    - "补齐 parser 缺失依赖"
  rationale: "collection.md 提取逻辑与 index 契约需完整设计"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S1-LLD.md"
---

# STORY-CR037-S1：topology_collection.py

## 目标

新建 `skills/topo-planning/src/topology_collection.py`，解析 `resource/network-topology/topology-collection.md` 提取 YAML 代码块，提供 `find_topology_file(topo_id)` 函数，支持 `index.yaml` 索引与 `compatible_consumers` 校验，从 interfaces 推导 `min_ports`。

## 依赖

无。

## 输出文件

`skills/topo-planning/src/topology_collection.py`

## AI 任务清单

- [ ] TASK-S1-01：实现 `find_topology_file(topo_id) → str`（YAML 文件路径）核心查找函数
- [ ] TASK-S1-02：实现 `_parse_collection_md(md_path) → dict[topo_id, yaml_dict]` 集合解析
- [ ] TASK-S1-03：实现 `_load_index(index_path) → dict` 索引加载与 tgfw-topo-collection 条目 compatible_consumers 校验（P0-2）
- [ ] TASK-S1-04：实现 `derive_min_ports(topo_dict) → dict[node_id, count]` 端口需求推导
- [ ] TASK-S1-05：补齐 parser.py 缺失依赖（`from topology_collection import find_topology_file` 可 import，统一 PYTHONPATH）
- [ ] TASK-S1-06：错误处理（collection.md 缺失 / topo_id 不在 collection key 集合 / YAML 解析失败 / tgfw-topo-collection 缺失或 compatible_consumers 不含 ptm-te）
- [ ] TASK-S1-07：为 topo_id 模糊匹配提供建议（编辑距离/前缀匹配；候选集 = `_parse_collection_md` 解析出的 12 个 node* topo_id，与 index.yaml 顶层无关，R3）
- [ ] TASK-S1-08：实现 `list_topologies()` 返回全部可用 topo_id（候选集 = `_parse_collection_md` key 集合，S1-A）

## 设计证据

见 `process/stories/STORY-CR037-S1-LLD.md`（14 章节 full-lld）。
