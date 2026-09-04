---
doc_type: IMPLEMENTATION
story_id: STORY-038-13
story_slug: multi-instance-trex
cr_id: CR-038
feature_id: F-CR038-E3
title: 多实例 TRex + interface_kind 透传（5 文件跨模块收口）— 实现执行证据
design_evidence_type: full-lld
lld_ref: process/stories/STORY-038-13-multi-instance-trex-LLD.md
source_hld: docs/design/HLD-CR-038.md §10.2 / §10.3 / §10.5
source_adr: docs/design/ARCHITECTURE-DECISION-CR-038.md §ADR-CR038-02 / §ADR-CR038-05
executed_by: meta-dev
executed_at: "2026-08-15"
---

# STORY-038-13 IMPLEMENTATION — 多实例 TRex + interface_kind 透传

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| Story `status=dev-ready` 且 `dev_gate=cp5-approved` | PASS | `process/stories/STORY-038-13.md` frontmatter |
| full-lld 设计证据已确认（CP5 approved） | PASS | `process/checkpoints/CP5-CR038.md` + LLD `status=lld-ready-for-review` |
| 依赖门控（`depends_on: STORY-038-02/03/06` hard） | PASS | 上游 CP6 `STORY-038-02/03/06-CODING-DONE.md` 已产出，pool_merge SW 归并 / physical_pool pppoe_server / exporter MVP+links 均可用 |
| 文件所有权无冲突（三 shared 文件 merge_owner，Wave 串行） | PASS | `file_ownership.merge_owner=STORY-038-13`；S02/S03/S06 已交付、无 `dev_running` 冲突 |
| AI 可执行任务清单存在 | PASS | LLD §8 TASK-E3-13-01~06 |
| 不修改已 approve 对象 / REQUIREMENTS / HLD / ADR / 蓝图三件套 | PASS | 仅改 5 个源码文件 + 新增 tests/fixtures |
| 安全约束（GE1_1~4 禁改动、凭据占位、pppoe 无 ip/next_hop=null、dry-run） | PASS | `_resolve_env` 拒绝凭据类变量；`_resolve_tg_instance` 只读；不真机下发 |

## 实现对象清单

| 对象 | 文件 | 变更类型 | 职责 |
|---|---|---|---|
| `InterfaceInfo.interface_kind`（默认 `'physical'`） | `skills/topo-config/src/topology_model.py` | 修改（primary） | DUT 接口类型透传（physical / pppoe，ADR-CR038-05） |
| `NodeInfo.trex_instance`（默认 `None`） | 同上 | 修改（primary） | TG 节点目标 TRex 实例名（GE/TE，ADR-CR038-02） |
| `_parse_interface` / `_parse_yaml_topology` 节点构造 | `skills/topo-config/src/parser.py` | 修改（primary） | 解析 `interface_kind` / `trex_instance` |
| `get_trex_instances` / `get_trex_instance_of_port` | `skills/topo-config/src/physical_pool.py` | 修改（shared, merge_owner） | `trex_instances` 块唯一真相源只读查询 + 接口级 instance 反查 |
| `get_free_ports` | 同上 | 修改 | 空闲端口 dict 透传 `instance` + `trex_port` |
| `UnifiedNode.trex_instances` / `UnifiedInterface.instance` | `skills/topo-planning/src/pool_merge.py` | 修改（shared, merge_owner） | 归并数据类扩展 |
| `_resolve_env` / `_resolve_instances_block` / `_merge_trex_instances` | 同上 | 新增 | `${TG_*}` 占位解析（非凭据）+ 按 name 归并（pool 侧为准） |
| `_merge_tg` / `_build_interfaces` / `to_pool_node` | 同上 | 修改 | 多实例归并接入 + 序列化 `trex_instances`/`instance` |
| `_resolve_tg_instance` | `skills/topo-planning/src/exporter.py` | 修改（shared, merge_owner） | 实例推导确定性决策表 |
| `_build_nodes` | 同上 | 修改 | 产出 `trex_instance`/`trex_sync_port`/`trex_async_port` + `trex_api_url` 优先级 |
| `_resolve_ip_plan` | 同上 | 修改 | pppoe 域不参与 next_hop（ADR-CR038-05） |
| `test_trex_multi_instance.py` | `skills/topo-config/tests/` | 新增 | 10 例（physical_pool + topology_model + parser） |
| `test_pool_merge_trex.py` | `skills/topo-planning/tests/` | 新增 | 10 例（pool_merge 多实例归并 + `_resolve_env`） |
| `test_exporter_trex.py` | `skills/topo-planning/tests/` | 新增 | 10 例（exporter 实例推导 + interface_kind） |
| `pool_trex_multi_instance.yaml` | `skills/topo-config/tests/fixtures/` | 新增 | physical_pool 多实例测试池 |
| `devices_trex_multi.yaml` | `skills/topo-planning/tests/fixtures/pool_merge/` | 新增 | devices.yaml tg 块 instances 补充块 |
| `pool_trex_multi.yaml` | `skills/topo-planning/tests/fixtures/` | 新增 | exporter 多实例测试池 |

## 设计契约映射

| Story 目标 / 需求 | TASK-ID | 实现对象 | 测试 |
|---|---|---|---|
| R-F-013（trex_instances 汇总 + instance 反查 + free_ports 透传） | TASK-E3-13-01 | physical_pool `get_trex_instances`/`get_trex_instance_of_port`/`get_free_ports` | `test_get_trex_instances_*` / `test_get_trex_instance_of_port_*` / `test_get_free_ports_passthrough` |
| R-F-014（pool 侧为准按 name 合并 + `_resolve_env` + 序列化） | TASK-E3-13-02 | pool_merge `_merge_tg`/`_merge_trex_instances`/`_resolve_env`/`to_pool_node` | `test_merge_tg_*` / `test_resolve_env_*` / `test_to_pool_node_*` |
| R-F-015（多实例导出 + 单值回退） | TASK-E3-13-03 | exporter `_resolve_tg_instance`/`_build_nodes` | `test_resolve_tg_instance_*` / `test_build_nodes_*` |
| R-F-016（interface_kind 透传） | TASK-E3-13-04/05 | topology_model `InterfaceInfo.interface_kind` + parser `_parse_interface` | `test_parser_interface_kind` / `test_interface_info_interface_kind_default` / `test_interface_kind_constructor_passthrough` |
| R-F-017（pppoe 无 ip、next_hop=None） | TASK-E3-13-03 | exporter `_resolve_ip_plan` | `test_interface_kind_pppoe_no_ip` / `test_interface_kind_physical_ip_checked` |
| R-NF-004（回归无破坏 / node2 单值回退） | 全部 | `_resolve_tg_instance` 无块回退分支 + `interface_kind` 缺省 physical | `test_build_nodes_node2_no_new_keys` / 既有 test_exporter.py 全绿 |
| ADR-CR038-02（physical_pool 唯一真相源） | TASK-E3-13-01/02 | `get_trex_instances` + `_merge_tg` 按 name 归并 | §5.1 链路 |
| ADR-CR038-05（pppoe 不产 ip） | TASK-E3-13-03/04/05 | `InterfaceInfo.interface_kind` 默认 physical + `_resolve_ip_plan` pppoe 跳过 | §5.2 链路 |
| ADR-09（9 占位不变，扩展 tg2.*） | 全部 | tg1/dut1 主实例键名不变，仅扩展 nodes.tg1 字段 | SCN-038-POS-010/011、BND-003 |

- 上游接口契约：S03 `get_pppoe_server`/`validate_pppoe_server`（physical_pool 基线）；S02 SW 归并（pool_merge 基线）；S06 `_validate_mvp_topology`/`_build_links`/`_build_port_mapping`（exporter 基线）。
- 扩展点闭环（OPEN-06-01）：S06 `_resolve_interface_kind` duck-typing `getattr(iface, 'interface_kind', None) or 'physical'`；本 Story 在 `InterfaceInfo` 加真实字段默认 `'physical'` → 零改动消费。

## 单元测试与 Fixture 计划

- 测试文件：`test_trex_multi_instance.py`（10 例）+ `test_pool_merge_trex.py`（10 例）+ `test_exporter_trex.py`（10 例），合计 30 例（LLD §7.1 全部覆盖）。
- 隔离策略：exporter 环回完整 ip_plan 仍受 `compute_ip_plan` 对 SW 拓扑 `IP_PLAN_SW_VLAN_MISSING` 限制，`test_interface_kind_pppoe_no_ip` / `test_interface_kind_physical_ip_checked` 手构 ip_plan + 直接调 `_resolve_ip_plan` 隔离；`test_interface_kind_constructor_passthrough` 用构造函数字段（非 setattr）验证 S06 duck-typing 零改动。
- Fixture：`pool_trex_multi_instance.yaml`（physical_pool 多实例池：tg-multi 含 trex_instances GE/TE + interfaces[port].instance，tg-single 无块）；`devices_trex_multi.yaml`（devices.yaml tg.instances 补充块）；`pool_trex_multi.yaml`（exporter 多实例池：单 TG + 单 DUT 直连）。
- 台账：测试内 `_fresh_ledger` 由池节点动态生成（device_id → brand='test' + 全空闲 port_status），无静态台账文件提交。
- 用例覆盖：POS-010（`test_build_nodes_multi_instance`）、POS-011（`test_interface_kind_pppoe_no_ip`）、BND-003（`test_build_nodes_node2_no_new_keys`）、R-F-013~017 单元级 + R-NF-004 回归。

## 最小实现切片

| 切片 | TASK-ID | 内容 | 完成准则 |
|---|---|---|---|
| S1 | TASK-E3-13-04 | `topology_model.py` `InterfaceInfo.interface_kind` + `NodeInfo.trex_instance` | `test_interface_info_interface_kind_default` / `test_node_info_trex_instance_default` |
| S2 | TASK-E3-13-05 | `parser.py` 解析 `interface_kind` / `trex_instance` | `test_parser_interface_kind` / `test_parser_trex_instance` |
| S3 | TASK-E3-13-01 | `physical_pool.py` 三接口 + free_ports 透传 | `test_get_trex_instances_*` / `test_get_trex_instance_of_port_*` / `test_get_free_ports_passthrough` |
| S4 | TASK-E3-13-02 | `pool_merge.py` 归并 + `_resolve_env` + 序列化 | `test_merge_tg_*` / `test_resolve_env_*` / `test_to_pool_node_*` |
| S5 | TASK-E3-13-03 | `exporter.py` `_resolve_tg_instance` + `_build_nodes` + `_resolve_ip_plan` | `test_resolve_tg_instance_*` / `test_build_nodes_*` / `test_interface_kind_*` |
| S6 | TASK-E3-13-06 | 单测 + fixture（POS-010/011、BND-003） | 全量 340 绿 |

## 平台差异处理

N/A —— 纯 Python 数据模型 / 归并 / 导出器改动，无平台路径、安装结构、CLI 或真机协议差异。不涉及 `delivery/doc/PLATFORM-CONTRACTS.yaml` / `PLATFORM-INSTALL-SPEC.md`。

## 验证结果

- 运行命令：`uv run --python 3.11 pytest skills/topo-config/tests/ skills/topo-planning/tests/`
- 结果：**340 passed, 0 failed**（基线 310 + 新增 30，无回归；仅 1 个既有 telnetlib DeprecationWarning）。
- 新增单测：`test_trex_multi_instance.py`（10）、`test_pool_merge_trex.py`（10）、`test_exporter_trex.py`（10）均通过。
- 回归：`test_exporter.py` / `test_exporter_loopback.py` / `test_pool_merge.py` / `test_pool_merge_sw.py` 全绿（S06 duck-typing + 单值回退行为不变，R-NF-004）。
- 场景：SCN-038-POS-010（多实例 env-file）PASS、SCN-038-POS-011（pppoe 无 ip/next_hop=None）PASS、SCN-038-BND-003（单实例回退 tg1/dut1）PASS。

## 未覆盖项

- 环回完整 ip_plan 端到端（`compute_ip_plan` 对 SW 拓扑抛 `IP_PLAN_SW_VLAN_MISSING`）——单测手构 ip_plan 隔离（沿用 RA-06-01 / OPEN-06-02，归属后续 Story/CR 供 vlan_assign）。
- 多实例真机 TRex 下发 / `trex_sync_port`/`trex_async_port` 协议消费——本 Story 仅数据模型 + env-file 导出，不真机下发（独立 runtime_authorization）。
- devices.yaml `instances` 补充块 schema 的正式固化——本 Story 按 LCQ-STORY-038-13-01 默认方案 A（嵌套 instances）实现，正式 schema 需 S08/集成期复核。
- 多环 / 分支拓扑——沿用 S06 `LOOPBACK_CLOSED_LOOP` 显式拦截（RA-038-004 首版）。

## 设计缺口反馈

1. **next_hop 语义澄清**：S06 LLD §5.2 明确「若 DUT 全部出口为 pppoe（无 L3 网关域），next_hop 自然保持 None」；本 Story `_resolve_ip_plan` next_hop 循环对 pppoe DUT 网关端口 `continue`，仅 physical 域参与，混合场景（physical + pppoe）取 physical 域网关、纯 pppoe 场景 next_hop=None（`test_interface_kind_pppoe_no_ip` 覆盖纯 pppoe 分支）。
2. **interface_kind 取值来源为逻辑拓扑节点级字段**：STORY 卡片目标「`_resolve_tg_instance` 接口级 `instance` 决定实例」的表述，落地为「节点级 `NodeInfo.trex_instance` 声明决定实例，接口级 `instance`（physical_pool）用于 free_ports 归属反查」；二者分工见 LLD §5.1/§5.3，无冲突。
3. **`_resolve_env` 凭据拒绝白名单**：`PASSWORD|TOKEN|SECRET` 大小写不敏感拒绝解析（LLD §10）；其余 `${TG_*}` 缺失保留原串 + warning（可降级）。

## 后续交接

- 验证入口：`uv run --python 3.11 pytest skills/topo-config/tests/ skills/topo-planning/tests/ -v`。
- 关键场景：SCN-038-POS-010（多实例 env-file）、SCN-038-POS-011（pppoe 无 ip/next_hop=None）、SCN-038-BND-003（单实例回退 tg1/dut1）。
- 风险提示：
  - 三 shared 文件（physical_pool.py/pool_merge.py/exporter.py）本 Story 为 merge_owner，已与 S02/S03/S06 交付基线兼容；后续改动需 merge_owner 协调。
  - OPEN-06-01 已闭环（S13 加真实 `interface_kind` 字段 + S06 duck-typing 零改动）；OPEN-06-02（环回完整 ip_plan 端到端）仍未闭环，归属后续 Story。
  - `compute_ip_plan` 对 SW 拓扑的 `IP_PLAN_SW_VLAN_MISSING` 限制仍在（单测手构 ip_plan 隔离）。
- 设计 delta：本 Story 无长期 Feature DESIGN / ADR / HLD 回写（`design_delta_required=false`）。
