---
doc_type: BLUEPRINT
id: BLUEPRINT-CR-038
cr_id: CR-038
parent_cr: CR-037
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（solution-design）
source_hld: docs/design/HLD-CR-038.md
---

# 蓝图 — CR-038（PPPoE 链路规划能力）

## 1. 蓝图适用性判定

| 触发条件 | 是否命中 | 判定 |
|---|---|---|
| 多个 Feature / Epic 需要明确边界、优先级或能力地图 | 是（5 Epic / 13 Story） | 需蓝图 |
| 存在领域对象、数据归属、状态流或持久化边界争议 | 是（SW 节点归属、PPPoE 数据源、多实例 TRex 归属） | 需蓝图 |
| 存在共享能力、复用层、跨 Feature 调用或依赖方向不清 | 是（topo_mapper/pool_merge/exporter 跨模块透传） | 需蓝图 |
| 存在跨 Feature 用户旅程、发布切片或回滚边界 | 是（6 切片 M1-M6） | 需蓝图 |

**结论：`required`**（非 N/A / waived）。本 CR 是 CR-037 的自然延续，跨 topo-config 引擎 + topo-planning 引擎 + 设备管理 + 执行编排 4 个能力域，必须先用蓝图固定 Feature 边界、数据归属与依赖方向，再进入 HLD / Story 拆解。

## 2. Feature / Epic 能力边界

| Epic | 能力 | 覆盖 Story | 数据归属 |
|---|---|---|---|
| E-038-1 SW 映射与台账 | SW 显式节点匹配 + 显式 SW 优先于自动透传 + 归并 | STORY-038-01/02 | topo_mapper / pool_merge；数据源 physical_pool.yaml + devices.yaml |
| E-038-2 PPPoE 配置 | Server 命令族（含数据源契约）+ Client 拨号 op | STORY-038-03/04/05 | switch_configurator / commands / op_mapper；数据源 physical_pool `pppoe_server` 块 |
| E-038-3 环回 env-file | exporter 突破 MVP + case_runner 兼容 + 多实例 TRex | STORY-038-06/07/13 | exporter / case_runner；数据源映射结果 + physical_pool `trex_instances` |
| E-038-4 SW 设备管理 | devices.yaml sw 块 + device-reference SW 别名 | STORY-038-08/09 | device-management 模板/参考 |
| E-038-5 验证与集成 | trex 双向发流 + 编排 + 拓扑建模 | STORY-038-10/11/12 | trex-traffic / ptm-te / topology-collection |

## 3. 数据归属（单一真相源）

| 数据对象 | 归属文件 | 说明 | 禁止重复定义 |
|---|---|---|---|
| SW 节点接线/端口/台账 | physical_pool.yaml + physical_ledger.yaml | node_type=SW 已预留（RA-038-005） | devices.yaml 不承载 SW 接线 |
| PPPoE ip pool 段 + local-user | physical_pool.yaml SW 节点 `pppoe_server` 块 | DQ-038-05，与 NGFW 拨号 163.0.0.x 同源 | 不重复定义于 devices.yaml/独立配置 |
| 多实例 TRex 汇总块 + 接口 instance | physical_pool.yaml TG 节点 `trex_instances` | R-F-013~015 | devices.yaml `tg.instances` 仅补充 port_map/ports/service |
| SW 设备块（host/brand） | devices.yaml `device_groups[].switch` | R-F-009 | physical_pool 不重复 brand（台账已有 brand） |
| SW 平台别名 | device-reference.md | R-F-010，唯一真相源 | 不在其他文档硬编码 |
| env-file 静态 IPAM | exporter 产物（ip_plan 权威 = ipam.ip_plan_to_dict） | 仅 TG/DUT 静态侧 | PPPoE 动态地址不进（R-C-006） |

## 4. 关键能力路由（能力 → 模块）

| 能力 | 模块 | 变更类型 |
|---|---|---|
| SW 显式匹配 + 显式优先 | skills/topo-config/src/topo_mapper.py | 修改 |
| SW 归并 | skills/topo-planning/src/pool_merge.py | 修改 |
| PPPoE 命令族 | skills/topo-config/src/commands.py | 修改 |
| PPPoE 下发 | skills/topo-config/src/switch_configurator.py | 修改 |
| PPPoE Client op 映射 | skills/case-execution 的 op_mapper（或 factor-library 侧） | 修改/增补 |
| MVP 放宽 + SW 节点 + 环回 | skills/topo-planning/src/exporter.py | 修改 |
| 多实例 TRex 消费 | skills/topo-config/src/physical_pool.py | 修改 |
| 接口模型扩展（interface_kind/instance/trex_instance） | skills/topo-config/src/topology_model.py + parser.py | 修改 |
| env-file SW 契约 | skills/case-execution/scripts/case_runner.py | 修改 |
| SW 设备 schema | skills/device-management/templates/devices.yaml.example | 修改 |
| SW 平台别名 | skills/device-management/reference/device-reference.md | 修改 |
| PPPoE 拓扑建模 | resource/network-topology/topology-collection.md | 修改 |
| 编排集成 | agents/ptm-te.md | 修改 |

## 5. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-se | 初稿：蓝图适用性=required + Epic 边界 + 数据归属 + 能力路由 |
