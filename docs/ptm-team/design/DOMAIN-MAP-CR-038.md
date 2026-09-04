---
doc_type: DOMAIN-MAP
id: DOMAIN-MAP-CR-038
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（solution-design）
---

# 领域映射 — CR-038（PPPoE 链路规划能力）

## 1. 领域对象

| 对象 | 字段（新增以 ▲ 标注） | 生命周期 | 归属 |
|---|---|---|---|
| UnifiedNode（归并节点） | node_id/node_type/device_id/device_type/hardware_platform/sub_type/api_server/management_*/interfaces/ledger_model/ledger_status/source + ▲trex_instances | 归并期（pool_merge 内存） | pool_merge.py |
| UnifiedInterface（归并端口） | port_id/physical_port/speed_class/media_type/trex_port/allocated_to/allocated_vlan + ▲instance | 归并期 | pool_merge.py |
| InterfaceInfo（逻辑接口） | logic_id/chassis_id/.../vrf/link + ▲interface_kind（默认 physical） | 解析期（parser）→ 映射 → 导出 | topology_model.py |
| NodeInfo（逻辑节点） | node_type/role/.../min_ports/node_types/layer/... + ▲trex_instance | 解析期 → 映射 | topology_model.py |
| DeviceMapping（设备映射） | logical/physical/device_id/node_type/role/device_type/status/reason/port_mappings | 映射期 | topo_mapper.py |
| LinkMapping（链路映射） | logical_link/endpoints/physical_cable/status/auto_inserted_switch | 映射期 | topo_mapper.py |
| pppoe_server 配置块 | ▲ip_pool_name/ip_pool_subnet/ip_pool_gateway/virtual_template/authentication/local_users[] | 台账（physical_pool.yaml SW 节点） | physical_pool.yaml |
| trex_instances 汇总块 | ▲{instance_name: {api_server, hardware_platform, sync_port?, async_port?}} | 台账（physical_pool.yaml TG 节点） | physical_pool.yaml |
| env-file nodes.swN | ▲host/brand/role/interfaces | 导出产物 | exporter.py |

## 2. 状态机

| 状态对象 | 状态值 | 流转 |
|---|---|---|
| DeviceMapping.status | matched / degraded / unmapped / virtual / auto_inserted | SW 显式匹配成功 → matched；Mock → virtual；自动透传 SW → auto_inserted；无候选 → unmapped（结构化 reason） |
| 台账 device.status | free / occupied / faulty | 显式 SW 映射成功 → occupied（复用 `_commit_allocation`）；释放 → free；faulty 不被覆盖 |
| 台账 port_status.allocated_to | null / env_name | 映射成功写 env_name；teardown/release 回 null |
| env-file metadata.executable | true / false | ip_plan 完整 → true；缺 IP 降级 → false（dry-run） |
| pppoe 接口 ip | 不产 ip（动态） | interface_kind=pppoe → 无 ip 字段、next_hop=null |

## 3. 核心规则（确定性）

1. **显式 SW 优先（R-F-018）**：自动透传候选 SW = 全部 SW − explicit_sw_reserved（显式 SW 候选集）− assigned_sws。
2. **SW env-file 契约（DQ-038-02）**：SW 进 `nodes.swN`，不进 `port_mapping`，不新增 `${ENV.sw.*}`。
3. **环回 links 无向（SGQ-038-02）**：links 去重（a-b 与 b-a 同端点对），方向由用例 txport/rxport 指定。
4. **地址池 vs IPAM（SGQ-038-01）**：PPPoE 动态地址不进静态 IPAM；ip_pool 段作为配置参数。
5. **VRF（SGQ-038-03）**：首版全局路由域，VRF 可选（复用 `ip binding vpn-instance`）。
6. **多实例归并（R-F-014）**：`_merge_tg` 按 name 合并，pool 侧 `trex_instances` 为准，补 devices 侧 `port_map/ports/service`。
7. **interface_kind 透传（R-F-016/017）**：默认 physical；pppoe 不产 ip、next_hop=None。
8. **凭据（ADR-02）**：`password_cipher`/管理密码一律 `${ENV_VAR}` 占位。

## 4. 术语表

| 术语 | 定义 |
|---|---|
| 显式 SW 节点 | 逻辑拓扑中 `node_type=SW` 的节点（区别于自动透传插入的 L2 SW） |
| 自动透传 | topo_mapper `_try_l2_pass_through` 在无直连 cable 时自动插入 L2 SW 的路径 |
| pppoe-server | `role=pppoe-server` 的 SW 节点，承载 PPPoE Server 配置 |
| 多实例 TRex | 同一 TG 主机并跑多个 TRex 实例（GE/TE），经 `trex_instances` + 接口 `instance` 表达 |
| interface_kind | dut1 接口类型（physical/pppoe），决定是否产静态 ip |
| 环回拓扑 | TG→SW→DUT→SW→SW→TG 的闭环链路，links 无向 |
| ip pool 段 | PPPoE Server 分配的拨号网段（163.0.0.x），与 NGFW 拨号同源 |

## 5. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-se | 初稿：领域对象/状态机/规则/术语 |
