---
doc_type: FEATURE-DESIGN
id: cr038-loopback-envfile
feature_id: F-CR038-E3
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
source_hld: docs/design/HLD-CR-038.md §10.3（exporter）+ §10.2（pool_merge）+ §10.5（topology_model/parser）
---

# Feature 设计 — 环回 env-file 导出（F-CR038-E3）

## 1. 目标与边界

- **目标**：突破单 TG + 单 DUT 直连 MVP，支持环回链路（link4: dut↔sw↔tg）生成 env-file；多实例 TRex 契约与 `interface_kind` 透传。
- **范围**：STORY-038-06（exporter）、STORY-038-07（case_runner）、STORY-038-13（多实例 TRex + interface_kind）。
- **非目标**：不改变 tg1/dut1 主实例 env 键；不处理多 DUT。

## 2. MVP 校验放宽（exporter._validate_mvp_topology）

- 合法：恰好 1 TG + 1 DUT + 任意 N SW；拒绝 Mock / PC / 多 DUT。
- `explicit_sw_reserved` 之外的 SW 允许作为环回中继。

## 3. links 环回校验

- 去重：同 `(port_a, port_b)` 仅一次。
- 自环：`port_a == port_b` → 结构化错误 `LOOPBACK_SELF_LOOP`。
- 闭环：`sw1:p1 ↔ sw1:p2` 或链路不连通 → 结构化错误 `LOOPBACK_CLOSED_LOOP`。
- 无有效环回 → 沿用单链路 env-file（不新增占位）。

## 4. interface_kind 透传（ADR-CR038-05）

- `_build_nodes` 中 `interface_kind` 默认 `'physical'`（现状），改为按端点节点类型透传：
  - 逻辑 `link4` 的 `tg.port1` 与 `dut.port1` 分别标记 `physical`；
  - SW 端接口 `interface_kind='pppoe'`（role=pppoe-server），pppoe 接口不产生 ip、`next_hop=null`。
- `_resolve_ip_plan` 对 pppoe 接口跳过 IP 计划完整性校验（pppoe 无 ip），其余接口仍校验 IP_PLAN_INCOMPLETE。

## 5. 多实例 TRex 契约（ADR-CR038-02，STORY-038-13）

- physical_pool 汇总 `trex_instances` 块为权威来源。
- pool_merge `_merge_tg`：按 name 归并，pool 侧 primary，补充 devices 侧 port_map/ports/service。
- exporter `_resolve_tg_instance`：接口级 `instance` 决定实例；未指定时单实例回退（现状 tg1/dut1）。
- env-file 键：多实例用 `tg2.port1` 等，主实例保持 `tg.port1`（ADR-09 9 占位不变）。

## 6. env-file 校验（case_runner.load_env_file，STORY-038-07）

- 保持校验 tg1/dut1；容忍额外 swN 节点，不做不必要拒绝。

## 7. 数据模型变更

| 对象 | 变更 |
|---|---|
| `UnifiedNode` | 新增 `trex_instance: str = 'tg1'`（默认主实例） |
| `UnifiedInterface` | 新增 `instance`（可选）；`interface_kind`（可选，默认 physical） |
| `InterfaceInfo` | 新增 `interface_kind`（可选，默认 physical） |
| `NodeInfo` | 新增 `trex_instance`（可选） |
| `parser._parse_interface` | 解析 `interface_kind`；`_resolve_node_type` 解析 `trex_instance` |

## 8. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-se | 初稿：MVP 放宽 + links 校验 + interface_kind + 多实例 TRex |
