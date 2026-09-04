---
doc_type: FEATURE-DESIGN
id: cr038-sw-mapping
feature_id: F-CR038-E1
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
source_hld: docs/design/HLD-CR-038.md §10.1/§10.2
---

# Feature 设计 — SW 映射与台账（F-CR038-E1）

## 1. 目标与边界

- **目标**：topo_mapper 支持 `node_type=SW` 显式逻辑节点匹配；以 reserved 集互斥保证显式 SW 优先于自动 L2 透传；`role=pppoe-server` 保留到 DeviceMapping；SW 归并（pool_merge）正确承载 SW 节点与台账映射。
- **范围**：STORY-038-01（topo_mapper）+ STORY-038-02（pool_merge）。
- **非目标**：不改变 Mock 语义（Mock 仍标 virtual）；不处理多 DUT。

## 2. 确定性算法：显式 SW 优先（R-F-018）

1. 回溯前预计算 `explicit_sw_reserved = { phys | phys ∈ 任一显式逻辑 SW 节点的物理候选 }`。
   - 候选来源：`_candidates_for_node` 对每个 `node_type=SW`（或 `'SW' in node_types`）逻辑节点返回的物理 SW 设备集合。
2. `_try_l2_pass_through(dev_a, dev_b, ...)` 候选 SW 集合 = `get_devices_by_type('SW')` − `explicit_sw_reserved` − `assigned_sws`。
   - `assigned_sws` = 回溯当前 `assignment.values()` 中 node_type=SW 的物理设备（现状已有）。
3. 显式 SW 节点按既有 `_backtrack` 正常匹配（node_type=SW 候选 + min_ports 筛选）。
4. 映射成功后 `_build_result` 保留 `role=node.role`（pppoe-server 等），并走 `_commit_allocation` 写台账。

**互斥不变量**：显式 SW 候选集 ∩ 自动透传候选集 = ∅，与回溯顺序无关。

## 3. role 传递

- `DeviceMapping(role=node.role)`：逻辑 SW 节点 `role=pppoe-server` → DeviceMapping.role 保留，供 switch_configurator 判定 PPPoE Server。
- 自动透传 SW 的 DeviceMapping 使用 `role='l2-pass-through'`（现状不变），与显式 role 区分。

## 4. 台账互斥与占用闭环

- 显式 SW 映射成功 → `_commit_allocation(used_ports, result, env_name)` 写 `port_status[port].allocated_to=env_name`。
- 自动透传 SW 端口同样进入 `used_ports`（现状 `_try_l2_pass_through` 返回 ports 已包含 SW 端口），复用占用闭环。
- reserved 集保证两者不冲突。

## 5. 失败路径（结构化 reason，R-NF-002）

| 场景 | DeviceMapping.status | reason |
|---|---|---|
| SW 无候选（pool 无 SW 节点） | unmapped | `无满足条件的空闲物理设备`（含降级建议） |
| SW 接口不匹配 / 连通性不满足 | unmapped | 连通性校验失败 reason |
| 显式 SW 与自动透传冲突 | 不触发（reserved 集已剔除） | — |

## 6. pool_merge SW 归并（STORY-038-02）

- `_merge_sw_pc` 已覆盖 SW（node_type in SW/PC/Mock），补充：SW 节点按 `management.host` 匹配 devices.yaml `switch` 块补充 device_type/hardware_platform（现状 `_merge_sw_pc` 调用处已有 `switch`/`client` 块查找）。
- 台账映射：`_resolve_ledger` 按 device_id 关联，SW 节点补齐 ledger_model/ledger_status 与 port_status 占用。

## 7. 接口契约

| 接口 | 变更 | 消费方 |
|---|---|---|
| `TopoMapper.match` | 预计算 explicit_sw_reserved 传入 `_try_l2_pass_through` | 内部 |
| `_try_l2_pass_through` | 新增 `explicit_sw_reserved` 参数（或实例属性） | 内部 |
| `DeviceMapping.role` | 不变（已存在），确保 SW role 传递 | exporter / switch_configurator |

## 8. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-se | 初稿：显式 SW reserved 互斥算法 + role 传递 + 台账互斥 |
