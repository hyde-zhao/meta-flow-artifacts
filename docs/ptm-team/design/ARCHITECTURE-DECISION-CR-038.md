---
doc_type: ARCHITECTURE-DECISION
id: ARCHITECTURE-DECISION-CR-038
cr_id: CR-038
parent_cr: CR-037
version: "0.1"
complexity: "medium"
confirmed: false
confirmed_by: ""
confirmed_at: ""
created_at: "2026-08-15"
owner: meta-se（solution-design）
---

# 架构决策 — CR-038（PPPoE 链路规划能力）

## 变更记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-se | 初稿：5 条核心 ADR（承接 CP2 七项决策的落地映射） |

## Agent/Skill 组合方案

| 阶段 | Agent | Skill/能力 |
|---|---|---|
| solution-design | meta-se | blueprint-design + hld-designer（本阶段） |
| story-planning | meta-se | phase-designer + dependency-mapper + wave-planner + story-manager + dag-validator（CP3 后） |
| 实现 | meta-dev | implementation-execution（full-lld / technical-note / waived） |
| 验证 | meta-qa | verification-execution + quality-review + release-readiness |

## 平台适配差异

| 差异点 | 处理 |
|---|---|
| H3C 真机（telnet） | 参考实现，真机下发独立 runtime_authorization |
| ruijie/huawei | 仅 dry-run diff 供人工核对（R-NF-001） |
| GE1_1~4 实例 | 禁改动，只操作 TE 实例与 TE2_1~4 |
| node2 单值向后兼容 | 无 trex_instances 回退单值 api_server |

## 核心 ADR

### ADR-CR038-01：显式 SW 节点与自动透传的避让关系

> 2026-08-17 修订（CR-038-FU-S01）：用户决定从「预计算 explicit_sw_reserved 全集排除」切换为「仅跳过已分配 SW」，已落地 topo_mapper `_try_l2_pass_through`（移除 `_compute_explicit_sw_reserved`）。

| 字段 | 内容 |
|---|---|
| 决策（当前实现） | 自动透传仅跳过 `assigned_sws`（已分配出去的 SW）；显式 SW 节点由 `_order_logical_nodes` 优先匹配，透传自然避让已占用 SW |
| 历史备选（已废弃） | 自动透传候选 SW = 全部 SW − explicit_sw_reserved（显式 SW 候选集）− assigned_sws（保守保留显式 SW 候选集） |
| 影响/风险 | topo_mapper `_try_l2_pass_through`；依赖回溯顺序（透传 link 先于显式 SW 匹配时可能抢占其候选） |
| 回退/切换 | 若发现透传抢占显式 SW 候选导致匹配失败，改回预计算 explicit_sw_reserved 全集排除 |

### ADR-CR038-02：多实例 TRex pool 侧为准按 name 合并

| 字段 | 内容 |
|---|---|
| 决策 | physical_pool `trex_instances` 为真相源，pool_merge `_merge_tg` 按 name 合并，补 devices 侧 port_map/ports/service |
| 备选 | devices.yaml `tg.instances` 唯一源 |
| 影响/风险 | physical_pool.py / pool_merge.py / exporter.py；不同源破坏台账同源 |
| 回退/切换 | 若实例改由 devices.yaml 唯一定义，调整合并方向 |

### ADR-CR038-03：PPPoE 数据源 = physical_pool `pppoe_server` 块

| 字段 | 内容 |
|---|---|
| 决策 | ip pool 段 + local-user（cipher 密文 `${ENV_VAR}`）存 physical_pool SW 节点 `pppoe_server` 块 |
| 备选 | devices.yaml / 独立配置 |
| 影响/风险 | R-F-019 读取路径；不同源破坏「同源一致」 |
| 回退/切换 | 若 ip pool 段跨设备共享改独立配置（DQ-038-05 切换条件） |

### ADR-CR038-04：环回 env-file 放行 = 1TG+1DUT+N SW + links 校验

| 字段 | 内容 |
|---|---|
| 决策 | `_validate_mvp_topology` 放宽为恰好 1 TG + 1 DUT + 任意 N SW；Mock/PC/多 DUT 仍拒绝；links 去重/自环/闭环校验 |
| 备选 | 任意节点组合全放行 / 独立环回导出器 |
| 影响/风险 | exporter.py；组合爆炸风险（RA-038-004 首版不穷举） |
| 回退/切换 | 多 DUT 需求出现时后续 CR 扩展 |

### ADR-CR038-05：interface_kind 透传 + pppoe 不产 ip

| 字段 | 内容 |
|---|---|
| 决策 | InterfaceInfo 增 interface_kind（默认 physical）；pppoe 接口不产 ip、next_hop=null |
| 备选 | 独立 pppoe 导出分支 |
| 影响/风险 | topology_model/parser/exporter；破坏静态 IPAM 语义（R-C-006 防线） |
| 回退/切换 | 若需静态引用拨号地址切方案 B（SGQ-038-01） |

## 设计确认点（需人工确认）

> 本 CR 的 7 项方向性决策（DQ-038-01/02/03/05 + SGQ-038-01/02/03）已在 CP2 approve。CP3 仅需确认本 ADR 5 条实现层映射（ADR-CR038-01~05）与 HLD 推荐方案，无新增用户决策项。

| 确认点 | 类型 | 推荐 |
|---|---|---|
| HLD 推荐方案（扩展 topo 引擎） | architecture | 采用方案 A |
| ADR-CR038-01~05 落地映射 | implementation | 接受 |
