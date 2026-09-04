---
doc_type: LLD
story_id: STORY-038-06
story_slug: exporter-loopback-mvp
cr_id: CR-038
feature_id: F-CR038-E3
title: exporter 突破 MVP + SW nodes + 环回 links 校验 — LLD
design_evidence_type: full-lld
lld_policy_required_level: full-lld
tier: S
risk_level: medium
batch_scope: n/a
homogeneous_story_pattern: n/a
shared_contract: n/a
source_hld: docs/design/HLD-CR-038.md §10.3
source_adr: docs/design/ARCHITECTURE-DECISION-CR-038.md §ADR-CR038-04 / §ADR-CR038-05
source_feature: docs/features/cr038-loopback-envfile/DESIGN.md
open_items:
  - S06/S13 interface_kind 语义边界（非阻断，见 §13.2，blocks_lld=false）
status: lld-ready-for-review
version: "0.1"
created_at: "2026-08-15"
owner: meta-dev
---

# STORY-038-06 LLD — exporter 突破 MVP + SW nodes + 环回 links 校验

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-dev | 初稿：MVP 放行规则（1TG+1DUT+N SW）+ SW nodes 导出 + links 去重/自环/闭环确定性算法 + interface_kind 透传起点 + S06/S13 边界 |

---

## 0. 工程依据

本 Story 是 F-CR038-E3（环回 env-file）的 exporter 核心，突破 exporter MVP「恰好 1 TG + 1 DUT」边界，支持 SW 节点导出与环回 links 校验。

- **HLD §10.3**：`exporter` 放宽为「恰好 1 TG + 1 DUT + 任意 N SW」，links 去重/自环/闭环校验，interface_kind 透传起点。
- **ADR-CR038-04**：环回放行 1TG+1DUT+N SW；**ADR-CR038-05**：interface_kind 透传（pppoe 接口不产 ip）。
- **Feature DESIGN（cr038-loopback-envfile）**：MVP 放行规则 + 环回 links 校验 + interface_kind 起点。
- **REQUIREMENTS**：R-F-006（MVP 放宽）、R-F-007（环回 links）。

**文档结构导览（14 段语义要点对齐）**：§1 目标与需求，§2 文件影响范围（代码结构），§3 接口设计（API），§4 数据模型，§5 核心流程与技术细节，§6 异常处理，§7 测试设计，§8 实施步骤，§9 回滚策略，§10 安全约束，§11 需求契约映射，§12 依赖与前置，§13 风险与开放项，§14 验收与交接（DoD）。

---

## 1. 背景与目标

### 1.1 Goal

在 `skills/topo-planning/src/exporter.py` 上增量兑现 HLD-CR-038 §10.3 与 ADR-CR038-04/05，将 env-file 导出的 MVP 边界从「恰好 1 TG + 1 DUT 直连」放宽为「恰好 1 TG + 1 DUT + 任意 N SW 环回」，并为 `interface_kind` 透传预留 S13 扩展点。

### 1.2 Functional Requirements（本 Story 范围）

| 编号 | 需求 | 量化验收 |
|------|------|---------|
| R-F-001 | `_validate_mvp_topology` 放宽 | 合法节点分布 = 恰好 1 TG + 恰好 1 DUT + 任意 N SW（N≥0）；TG≠1、DUT≠1 或含 Mock/PC/UNKNOWN/MULTI/其他类型 → `TOPO_NODE_COUNT_UNSUPPORTED`（见 §6.1） |
| R-F-002 | SW nodes 导出 | 每个显式 SW 物理设备在 env-file `nodes` 下产出 1 个 `swN` 节点，含 `host`/`brand`/`role`/`interfaces`（`interfaces` 恒为空 dict）；SW **不进入** `port_mapping` |
| R-F-003 | links 去重 | 环回 links 按无向端点对 `frozenset({(node_a, iface_a), (node_b, iface_b)})` 去重，同一无向对只输出一次 |
| R-F-004 | links 自环检测 | 端点 `(node, iface)` 完全相同（端口指向自身）→ `LOOPBACK_SELF_LOOP` |
| R-F-005 | links 闭环检测 | 同一 SW 节点两端口直连（`sw1:p1 ↔ sw1:p2`），或链路不连通/存在度数 ≠ 2 的节点（未构成单一简单环）→ `LOOPBACK_CLOSED_LOOP` |
| R-F-006 | `interface_kind` 透传起点 | DUT 接口 `interface_kind` 按逻辑接口读取，缺省 `'physical'`；值为 `'pppoe'` 的 DUT 接口不产 ip 且豁免 IP 完整性校验 |
| R-F-007 | `_resolve_ip_plan` pppoe 跳过 | `interface_kind='pppoe'` 的 DUT 接口跳过 IP 填充与完备性校验；`physical` 接口仍校验 `IP_PLAN_INCOMPLETE` |

### 1.3 Non-Functional Requirements

| 编号 | 约束 | 设计响应 |
|------|------|---------|
| R-NF-001 | 回归无破坏 | `N SW == 0` 时保持单链路模式既有行为逐字节不变（node2 系列既有测试全绿，见 §7.3） |
| R-NF-002 | 确定性 | 固定输入（含固定 `generated_at`）两次导出产物逐字节一致；SW 节点按 `dm.logical` 升序分配 `sw1/sw2/...`；臂配对按 TG 接口名升序分配 `port1/port2` |
| R-NF-003 | 结构化失败 | 新增错误走 `ExporterError(code, details)` 统一格式，`code` 为机器可枚举字符串，`details` 含 `topology_id` + 定位信息 |
| 安全 | 不触碰凭据 | 本 Story 纯导出器改动，无真机下发、无凭据读写；沿用既有 `_CREDENTIAL_RE` 凭据拦截（见 §10） |

### 1.4 模块拆分与职责

| 模块/函数 | 职责 | 本 Story 是否改动 |
|-----------|------|-------------------|
| `_validate_mvp_topology` | 节点分布合法性判定（放宽到 SW） | 改（TASK-E3-06-01） |
| `_validate_loopback_links` | 环回 links 去重/自环/闭环校验（新增） | 新增（TASK-E3-06-02） |
| `_pair_loopback_arms` | 环回臂配对（TG 端口 → DUT 端口，SW 中继透明）（新增） | 新增（TASK-E3-06-02） |
| `_build_port_mapping` | port_mapping 构建（单链路/环回双模式） | 改（TASK-E3-06-02） |
| `_build_nodes` | tg1/dut1/swN 节点 + `interface_kind` 透传 | 改（TASK-E3-06-03） |
| `_resolve_interface_kind` | duck-typing 读 `interface_kind`，缺省 physical（新增） | 新增（TASK-E3-06-03） |
| `_resolve_ip_plan` | pppoe 接口跳过 IP 填充/校验 | 改（TASK-E3-06-04） |
| `_build_links` | 环回折叠臂 links 输出 | 改（TASK-E3-06-02） |
| `_check_link_count` | link 数一致性（环回仍适用，不改） | 不改 |

---

## 2. 文件影响范围

### 2.1 代码结构（本 Story 写入范围）

| 文件 | 所有权 | 变更类型 |
|------|--------|---------|
| `skills/topo-planning/src/exporter.py` | `shared`（与 STORY-038-13 共享，S13 为 `merge_owner`，本 Story 为首次写入方） | 修改：`_validate_mvp_topology` / `_build_port_mapping` / `_build_nodes` / `_build_links` / `_resolve_ip_plan`；新增 `_validate_loopback_links` / `_pair_loopback_arms` / `_resolve_interface_kind` |
| `skills/topo-planning/tests/test_exporter_loopback.py` | `primary`（本 Story 独有新增文件） | 新增：环回单测（见 §7） |
| `skills/topo-planning/tests/fixtures/loopback_*.yaml` | `primary`（本 Story 独有新增文件） | 新增：环回拓扑/ip_plan fixture |

### 2.2 明确不改动的文件（所有权约束）

| 文件 | 原因 |
|------|------|
| `skills/topo-config/src/topology_model.py` | `InterfaceInfo.interface_kind` 字段由 STORY-038-13 添加（S13 primary） |
| `skills/topo-config/src/parser.py` | `interface_kind` 解析由 STORY-038-13 添加（S13 primary） |
| `skills/topo-planning/src/pool_merge.py` | 由 STORY-038-02 产出，本 Story 只读消费 |
| `skills/topo-config/src/physical_pool.py` | `trex_instances` / `get_brand` 由 STORY-038-13 扩展，本 Story 只读调用既有 `get_management`/`get_brand` |
| 已 approve 对象（`HLD-CR-038.md`/`ARCHITECTURE-DECISION-CR-038.md`/蓝图三件套/REQUIREMENTS） | 门禁约束，禁止修改 |

### 2.3 S06/S13 共享文件串行约束

`exporter.py` 由 STORY-038-06 与 STORY-038-13 共享，S13 为 `merge_owner`。本 Story 作为首次写入方，须：

1. 不占用 S13 的扩展点命名（`_resolve_tg_instance`、`trex_instance`/`trex_sync_port`/`trex_async_port` 均归 S13，本 Story 不新增）。
2. `interface_kind` 读取采用 duck-typing（`getattr(iface, 'interface_kind', 'physical')`），使 S13 加字段后本 Story 代码无需再改，构成向后兼容扩展点（见 §3.7、§13.2）。

---

## 3. 接口设计

> 本章只定义本 Story 涉及的函数签名、输入/输出契约、失败路径。实现对象（代码）在 CP6 阶段产出，本 Story 仅产出设计证据。

### 3.1 `_validate_mvp_topology`

```text
_validate_mvp_topology(logical_topo: Topology) -> Optional[str]
```

- 输入：逻辑拓扑（`logical_topo.nodes` / `logical_topo.metadata.topology_id`）。
- 输出：`None`（通过）；否则返回确定性错误消息字符串（由调用方包装为 `ExporterError('TOPO_NODE_COUNT_UNSUPPORTED', msg)`）。
- 放行规则（确定性）：
  1. 逐节点取类型集合 `types = set(node.node_types) or ({node.node_type} if node.node_type else {'UNKNOWN'})`。
  2. 统计 `tg_count = |{n: 'TG' in types(n)}|`、`dut_count = |{n: 'DUT' in types(n)}|`、`sw_count = |{n: 'SW' in types(n)}|`。
  3. 非法类型判定：任一节点的 `types` 存在不在 `{'TG','DUT','SW'}` 内的类型（含 `Mock`/`PC`/`UNKNOWN`）或 `|types| > 1`（`MULTI`）→ 拒绝。
  4. 放行当且仅当：`tg_count == 1` 且 `dut_count == 1` 且所有节点类型 ⊆ `{'TG','DUT','SW'}`（`sw_count ≥ 0`，**无硬上限**，上限由 §6.3 闭环校验约束）。
- 拒绝理由覆盖：多 DUT（`dut_count ≥ 2`）、多 TG（`tg_count ≥ 2`）、含 Mock/PC、含 UNKNOWN、含 MULTI。

### 3.2 `_validate_loopback_links`（新增）

```text
_validate_loopback_links(logical_topo: Topology, mapping_result: MappingResult) -> list[Arm]
```

- 前置：仅在环回模式（`N SW > 0`）调用；单链路模式（`N SW == 0`）不调用。
- 输入：逻辑拓扑 + 映射结果（取 `link_mappings` 中 `status == 'matched'` 且 `auto_inserted_switch` 为空的 link）。
- 输出：去重后的环回边集（供 `_pair_loopback_arms` 消费）。
- 失败：抛 `ExporterError('LOOPBACK_SELF_LOOP' | 'LOOPBACK_CLOSED_LOOP', details)`（见 §6.2/§6.3）。
- 算法见 §5.3。

### 3.3 `_pair_loopback_arms`（新增）

```text
_pair_loopback_arms(logical_topo: Topology, mapping_result: MappingResult, ring_edges: list) -> list[dict]
```

- 输出：有序臂列表 `[{port_key, tg_iface, dut_iface, tg_phys, dut_phys}, ...]`，长度恒等于 2（`port1`/`port2`），按 TG 接口名升序。
- `tg_phys`/`dut_phys`：由 TG/DUT `DeviceMapping.port_mappings` 中 `logical_port == tg_iface/dut_iface` 反查 `physical_port`。
- 算法见 §5.4。

### 3.4 `_build_port_mapping`（修改）

```text
_build_port_mapping(mapping_result, logical_topo) -> (port_mapping: dict, port_by_link: dict, arm_map: dict)
```

- 单链路模式（`N SW == 0`）：既有行为不变，返回值扩展第三项 `arm_map = {}`（空，向后兼容）。
- 环回模式（`N SW > 0`）：调用 `_validate_loopback_links` + `_pair_loopback_arms`，产出 `port_mapping = {port1: {tg, dut}, port2: {tg, dut}}`，`arm_map = {port1: {tg_iface, dut_iface}, port2: {tg_iface, dut_iface}}`，`port_by_link` 由臂映射填充（`port_key` 即臂键）。
- 失败：`LOOPBACK_SELF_LOOP` / `LOOPBACK_CLOSED_LOOP` / `PORT_MAPPING_INCOMPLETE`（TG/DUT 侧缺物理端口映射）。

### 3.5 `_resolve_interface_kind`（新增）

```text
_resolve_interface_kind(iface: object) -> str
```

- 语义：`return getattr(iface, 'interface_kind', None) or 'physical'`。
- 原因：`InterfaceInfo.interface_kind` 字段由 S13 添加到 `topology_model.py`，本 Story 用 duck-typing 读取，S13 加字段后无需再改（扩展点预留，见 §13.2）。

### 3.6 `_build_nodes`（修改）

- 新增 SW 节点（环回模式）：按 `dm.logical` 升序遍历 `status == 'matched'` 且 `node_type` 含 `SW` 的 `DeviceMapping`，产出：

```yaml
swN:
  host: <pool.get_management(sw_phys).get('host')>   # 无则 null
  brand: <pool.get_brand(sw_phys)>                   # 无则 null
  role: <dm.role>                                    # 如 'pppoe-server'，无则 null
  interfaces: {}                                     # 恒为空 dict，SW 不进 port_mapping
```

- DUT 接口 `interface_kind`：由硬编码 `'physical'` 改为 `_resolve_interface_kind(logical_topo.nodes[dut_node].interfaces[dut_iface])`（`dut_iface` 来自 `arm_map` 或既有 `port_by_link` 反查）。
- TG 接口 schema 不变（`{ip, gw}`），不新增 `interface_kind`（TG 接口始终按 physical 处理，见 §13.2）。

### 3.7 `_resolve_ip_plan`（修改）

- 判断粒度：`nodes['dut1']['interfaces'][port_key]['interface_kind']`。
- pppoe 跳过范围：
  1. IP 填充：`interface_kind == 'pppoe'` 的 DUT 接口不写入 `ip`（保持 `None`），且该接口对应的 `ip_plan` 反查命中也不写入。
  2. 完备性校验：`interface_kind == 'pppoe'` 的 DUT 接口豁免「DUT 侧缺 IP → `IP_PLAN_INCOMPLETE`」检查。
- TG 接口行为不变（仍按 physical 校验 `IP_PLAN_INCOMPLETE`）。
- `next_hop`：既有逻辑不变（从 DUT 业务出口广播域网关取 `.254` 去掩码）；若 DUT 全部出口为 pppoe（无 L3 网关域），`next_hop` 自然保持 `None`（ADR-CR038-05）。

---

## 4. 数据模型

### 4.1 env-file `nodes.swN` 结构（新增）

| 字段 | 类型 | 来源 | 缺省 |
|------|------|------|------|
| `host` | `str \| null` | `pool.get_management(sw_phys)['host']` | `null` |
| `brand` | `str \| null` | `pool.get_brand(sw_phys)` | `null` |
| `role` | `str \| null` | `dm.role`（STORY-038-01 已保留，如 `pppoe-server`） | `null` |
| `interfaces` | `dict` | 恒为 `{}` | `{}` |

- **不新增** `${ENV.sw.*}` 占位符（DQ-038-02 / ADR-09 九类占位不变）。
- **不新增** `port_mapping` 中的 SW 侧（HLD §12.4「port_mapping 仅 tg/dut」）。

### 4.2 `interface_kind` 透传（S06 起点）

| 位置 | 字段 | 缺省 | 说明 |
|------|------|------|------|
| `nodes.dut1.interfaces.portN.interface_kind` | `str` | `'physical'` | 由逻辑 DUT 接口 duck-typing 读取；`'pppoe'` 时不产 ip |

- S06 **不** 修改 `InterfaceInfo`/`NodeInfo` 数据模型（归 S13）；S06 仅读取 duck-typed 属性。
- 无持久化变更（本 Story 纯内存导出，不写台账/数据库）。

---

## 5. 核心流程

### 5.1 双模式分派

| 模式 | 判定 | 行为 |
|------|------|------|
| 单链路模式 | `sw_count == 0` | 完全沿用 CR-037 既有流程（`_build_port_mapping`/`_build_nodes`/`_build_links` 行为不变） |
| 环回模式 | `sw_count > 0` | 启用 §5.3 links 校验 + §5.4 臂配对 + §3.6 swN 节点 + §3.7 pppoe 跳过 |

`export_env_file` 主流程（新增步骤以 `[NEW]` 标注）：

```text
1. _validate_mvp_topology(logical_topo)                    → TOPO_NODE_COUNT_UNSUPPORTED（放宽后）
2. _check_link_count(mapping_result, logical_topo)          → LINK_COUNT_MISMATCH（不变，环回仍 3==3 通过）
3. [NEW] 若 sw_count > 0：_validate_loopback_links(...)     → LOOPBACK_SELF_LOOP / LOOPBACK_CLOSED_LOOP
4. ip_plan 完备性预检（ip_plan is None → IP_PLAN_REQUIRED / allow_degraded 降级）
5. _build_metadata(...)
6. _build_port_mapping(...)  → (port_mapping, port_by_link, arm_map)   [单链路/环回双模式]
7. _build_nodes(..., arm_map) → tg1/dut1/swN + interface_kind
8. _build_links(...)          → 单链路：既有 links；环回：折叠臂 links
9. 凭据检查（ADR-02）→ 原子落盘
```

### 5.2 节点类型解析（放宽后的确定性规则）

```text
_types(node):
    nts = getattr(node, 'node_types', None) or ([getattr(node, 'node_type', None)] if getattr(node, 'node_type', None) else [])
    return set(nts) or {'UNKNOWN'}
```

- `'SW' in types` → SW 节点；`'TG' in types` → TG；`'DUT' in types` → DUT。
- `|types| > 1` 或含 `Mock/PC/UNKNOWN/其他` → 非法（拒绝）。

### 5.3 links 校验算法（确定性，TASK-E3-06-02）

输入：环回模式下的 matched 非 auto_inserted link 集 `R`。

```text
1. 去重：以 frozenset({(node_a, iface_a), (node_b, iface_b)}) 为键，同一无向对保留首次，产出边集 E。
2. 自环：对每条 link，若 (node_a, iface_a) == (node_b, iface_b) →
        ExporterError('LOOPBACK_SELF_LOOP', f"环回拓扑存在自环 link '{lid}'：{node}:{iface} 指向自身")
3. 同节点异端口：若 node_a == node_b 且 iface_a != iface_b →
        ExporterError('LOOPBACK_CLOSED_LOOP', f"环回拓扑存在闭合子环 link '{lid}'：{node}:{iface_a} ↔ {node}:{iface_b}")
4. 闭环完整性：以 E 建无向图 G（节点粒度，忽略端口）：
   a. 连通分量数 != 1 → ExporterError('LOOPBACK_CLOSED_LOOP', f"环回链路不连通：连通分量数={k}")
   b. 存在节点度数 != 2 → ExporterError('LOOPBACK_CLOSED_LOOP', f"环回未构成单一简单环：节点 {node} 度数={d}（期望 2）")
5. 全部通过 → 返回 E（合法单一简单环）。
```

### 5.4 臂配对算法（确定性，TASK-E3-06-02）

输入：去重后边集 `E`、TG/DUT 节点定位。

```text
1. 由 E 建端口级邻接表 adj: (node, iface) -> (neighbor_node, neighbor_iface)。
2. 取 TG 节点的 2 个 ring 端口 {p_a, p_b}（按接口名升序）。
3. 对每个 TG 端口 p：
   cur = adj[(tg, p)]
   while cur.node 是 SW：
       sw_other = SW 的另一个 ring 端口（≠ cur.iface，SW 度恒为 2）
       nxt = adj[(cur.node, sw_other)]
       if nxt.node 是 DUT：return nxt.iface
       if nxt.node 是 TG：raise LOOPBACK_CLOSED_LOOP（防御性，正常已由 §5.3 拦截）
       cur = nxt    # 继续跨 SW 链
   if cur.node 是 DUT：return cur.iface
   raise LOOPBACK_CLOSED_LOOP（防御性）
4. 产出 2 个 (tg_iface, dut_iface) 配对，按 TG 接口名升序分配 port1/port2。
5. port_mapping[portN] = {tg: 物理端口(tg_iface), dut: 物理端口(dut_iface)}。
```

### 5.5 links 输出（环回折叠臂）

环回模式 `_build_links` 输出 2 条折叠臂（SW 透明，不引用 sw 节点）：

```text
links = [
    {from: f'{tg_node}.port1', to: f'{dut_node}.port1', name: 'port1'},
    {from: f'{tg_node}.port2', to: f'{dut_node}.port2', name: 'port2'},
]
```

- `name` 取 `port_key`（折叠臂无单一逻辑 link id，与单链路 `name=lid` 的差异见 §13.2 取舍记录）。
- `tg_node`/`dut_node` 为逻辑节点 id（与 `_build_nodes` 固定键 `tg1`/`dut1` 一致）。

---

## 6. 异常处理

### 6.1 `TOPO_NODE_COUNT_UNSUPPORTED`（放宽后）

- 触发：`_validate_mvp_topology` 不满足 §3.1 放行规则。
- `details` 格式：`topology '{topo_id}' 超出 MVP 边界：节点分布 {counts}。合法 = 1 TG + 1 DUT + N SW；拒绝 Mock/PC/多 DUT/多 TG/UNKNOWN/MULTI。`
- 与既有错误码兼容（调用方无需区分新旧，`code` 不变）。

### 6.2 `LOOPBACK_SELF_LOOP`（新增）

| 字段 | 值 |
|------|-----|
| 触发 | 端口指向自身：`(node, iface) == (node, iface)` |
| `code` | `LOOPBACK_SELF_LOOP` |
| `details` | 含 `link_id` + `node` + `iface` 定位 |

### 6.3 `LOOPBACK_CLOSED_LOOP`（新增）

| 字段 | 值 |
|------|-----|
| 触发 | (a) 同 SW 节点两端口直连 `sw1:p1 ↔ sw1:p2`；(b) 连通分量数 ≠ 1；(c) 存在度数 ≠ 2 的节点 |
| `code` | `LOOPBACK_CLOSED_LOOP` |
| `details` | 按子类型分别含 `闭合子环 link '{lid}'` 或 `连通分量数={k}` 或 `节点 {node} 度数={d}` |

### 6.4 `IP_PLAN_INCOMPLETE`（pppoe 豁免）

- 既有行为：任一 port 的 TG/DUT 接口缺 IP → `IP_PLAN_INCOMPLETE`。
- 本 Story 变更：`interface_kind == 'pppoe'` 的 DUT 接口豁免 DUT 侧检查（TG 侧仍校验）。

### 6.5 错误结构统一约定

所有新增错误沿用 `ExporterError(code, details)`：`code` 为机器可枚举字符串，`details` 为人读定位信息，不得包含凭据（§10）。

---

## 7. 测试设计

### 7.1 测试对象与入口

- 测试文件：`skills/topo-planning/tests/test_exporter_loopback.py`。
- 验证入口：`uv run --python 3.11 pytest skills/topo-planning/tests/`。
- 隔离策略：S06 单测用**手构 `ip_plan`** + **duck-typed `interface_kind` 注入**（在 `NodeInfo` 实例上 `setattr(node.interfaces[p], 'interface_kind', 'pppoe')`），绕开 S13 的 parser/topology_model 未就绪与 `compute_ip_plan` 对 SW 拓扑的 `IP_PLAN_SW_VLAN_MISSING` 限制（见 §13.3）。

### 7.2 用例清单（覆盖 SCN-038-POS-006 / BND-004 / NEG-005）

| 用例 ID | 场景 | 断言要点 |
|---------|------|---------|
| T-06-01 | POS-006 环回导出 | `nodes` 含 `tg1`/`dut1`/`sw1`；`port_mapping` 仅含 `port1`/`port2`（无 sw 键）；`sw1.interfaces == {}`；`links` 为 2 条折叠臂 `tg1.port1↔dut1.port1`、`tg1.port2↔dut1.port2` |
| T-06-02 | POS-006 三层结构 | `sw1.host`/`sw1.brand`/`sw1.role` 由 pool/dm 正确填充；`sw1.role == 'pppoe-server'` |
| T-06-03 | BND-004 Mock 拒绝 | 1 TG + 1 DUT + 1 Mock → `TOPO_NODE_COUNT_UNSUPPORTED` |
| T-06-04 | BND-004 PC 拒绝 | 1 TG + 1 DUT + 1 PC → `TOPO_NODE_COUNT_UNSUPPORTED` |
| T-06-05 | BND-004 多 DUT 拒绝 | 1 TG + 2 DUT → `TOPO_NODE_COUNT_UNSUPPORTED` |
| T-06-06 | BND-004 SW 放行 | 1 TG + 1 DUT + 1 SW → `_validate_mvp_topology` 返回 `None` |
| T-06-07 | BND-004 多 SW 放行 | 1 TG + 1 DUT + 2 SW 链 → 放行（N 无硬上限） |
| T-06-08 | NEG-005 自环 | 构造端口自环 link → `LOOPBACK_SELF_LOOP` |
| T-06-09 | NEG-005 同节点异端口 | `sw1:p1 ↔ sw1:p2` → `LOOPBACK_CLOSED_LOOP` |
| T-06-10 | NEG-005 链路不连通 | 环回断裂（度 1 dangling）→ `LOOPBACK_CLOSED_LOOP` |
| T-06-11 | NEG-005 去重 | 同无向对重复 link → links 只输出一次 |
| T-06-12 | interface_kind 透传 | DUT 接口缺省 `interface_kind == 'physical'`；注入 `'pppoe'` 后 `nodes.dut1.interfaces.portN.interface_kind == 'pppoe'` 且 `ip is None` |
| T-06-13 | pppoe 跳过 IP 校验 | 注入 pppoe 后 `_resolve_ip_plan` 不因该接口缺 IP 抛 `IP_PLAN_INCOMPLETE`；physical 接口缺 IP 仍抛 |
| T-06-14 | 回归（单链路） | `N SW == 0` 时既有 node2 系列导出行为不变（复用既有 `test_exporter.py` 全绿） |

### 7.3 回归与集成

- 既有 `test_exporter.py` 全部用例保持通过（单链路模式行为不变）。
- 环回产物经 `case_runner.load_env_file`/`build_env_topology` 结构校验通过（`port1`/`port2` + `tg1`/`dut1` + `links` 非空），复用既有集成测试模式（`test_exporter.py::test_load_env_file_accepts_export`）。

---

## 8. 实施步骤

| 步骤 | TASK-ID | 内容 | 输出 | 完成准则 |
|------|---------|------|------|---------|
| 1 | TASK-E3-06-01 | `_validate_mvp_topology` 放宽为 1 TG + 1 DUT + N SW，拒绝 Mock/PC/多 DUT/多 TG/UNKNOWN/MULTI | `exporter.py` | T-06-03~T-06-07 通过 |
| 2 | TASK-E3-06-02 | 新增 `_validate_loopback_links` + `_pair_loopback_arms`；`_build_port_mapping`/`_build_links` 环回双模式 | `exporter.py` | T-06-01/T-06-08~T-06-11 通过 |
| 3 | TASK-E3-06-03 | `_build_nodes` 新增 swN 节点 + `_resolve_interface_kind` duck-typing 透传 | `exporter.py` | T-06-01/T-06-02/T-06-12 通过 |
| 4 | TASK-E3-06-04 | `_resolve_ip_plan` pppoe 接口跳过 IP 填充与完备性校验 | `exporter.py` | T-06-13 通过 |
| 5 | TASK-E3-06-05 | 环回单测 + fixture（POS-006/BND-004/NEG-005） | `tests/` | 全量 `pytest skills/topo-planning/tests/` 绿 |

- 实现顺序：TASK-E3-06-01 → 02 → 03 → 04 → 05（步骤 5 与 1~4 同步推进，每步局部验证）。
- 每个 TASK-ID 与 §2.1 文件影响范围一一对应。

---

## 9. 回滚策略

| 回滚粒度 | 动作 |
|----------|------|
| 单 TASK 回退 | 每个 TASK 独立可回退：`_validate_mvp_topology`/`_build_nodes`/`_resolve_ip_plan` 按函数级增量，出错时 `git revert` 对应 commit |
| 环回模式回退 | `N SW == 0` 分支与环回分支物理隔离（`if sw_count > 0:` 分派），关闭环回即恢复单链路既有行为，零残留 |
| 全量回退 | 恢复 `exporter.py` 到 CR-037 基线，删除新增 `test_exporter_loopback.py` + `fixtures/loopback_*.yaml`，既有 node2 系列回归不受影响 |
| 数据模型回退 | 本 Story 无数据模型/台账变更，无需迁移回滚 |

- 回退触发：CP7 判定 `NEEDS_REWORK`/`NEEDS_DESIGN_CLARIFICATION`，或既有 node2 系列回归失败。
- 回退后验证：`uv run --python 3.11 pytest skills/topo-planning/tests/test_exporter.py` 全绿。

---

## 10. 安全约束

| 约束 | 设计响应 |
|------|---------|
| 凭据禁止（ADR-02） | 不新增凭据字段；沿用 `_CREDENTIAL_RE` 对产物全文正则拦截，环回路径不绕过 |
| 无真机下发 | 本 Story 仅 env-file 导出，无 telnet/SSH/写台账；SW 节点仅读 `get_management`/`get_brand` 元数据 |
| 无组合爆炸（RA-038-004） | 不穷举节点组合；放行规则仅按节点类型集合判定，不做排列验证（首版） |
| 错误信息无凭据泄漏 | `ExporterError.details` 只含 `topology_id`/`node`/`iface`/`link_id`，不含 host/密码/token |

---

## 11. 契约映射

| Story 目标 | TASK-ID | 文件 | 测试 |
|-----------|---------|------|------|
| `_validate_mvp_topology` 放宽（1TG+1DUT+N SW） | TASK-E3-06-01 | `exporter.py::_validate_mvp_topology` | T-06-03~T-06-07 |
| links 去重/自环/闭环校验 | TASK-E3-06-02 | `exporter.py::_validate_loopback_links`/`_pair_loopback_arms` | T-06-08~T-06-11 |
| `_build_nodes` SW 节点 + interface_kind 透传 | TASK-E3-06-03 | `exporter.py::_build_nodes`/`_resolve_interface_kind` | T-06-01/T-06-02/T-06-12 |
| `_resolve_ip_plan` pppoe 跳过 | TASK-E3-06-04 | `exporter.py::_resolve_ip_plan` | T-06-13 |
| 单测 | TASK-E3-06-05 | `tests/test_exporter_loopback.py` + fixtures | 全量绿 |

- 契约来源：HLD-CR-038 §10.3（exporter 修改）、ADR-CR038-04（放行规则）、ADR-CR038-05（interface_kind + pppoe 不产 ip）、Feature DESIGN §2/§3/§4、DQ-038-02（SW 进 nodes 不进 port_mapping）。
- 上游接口契约：STORY-038-01 保证 `DeviceMapping.role` 已保留（消费于 `swN.role`）；STORY-038-02 保证 SW 归并结果（消费于 SW 识别）。

---

## 12. 依赖与前置

| 依赖 | 类型 | 门控 | 本 Story 消费 |
|------|------|------|---------------|
| STORY-038-01（SW 显式映射） | hard | 上游 `verified` | `DeviceMapping.role`（`swN.role`）、SW `status=matched` |
| STORY-038-02（SW 归并） | hard | 上游 `verified` | SW 归并结果、SW 节点识别 |

- 前置检查（进入实现前必须满足）：
  1. Story `status == dev-ready` 且 `design_evidence_confirmed == true`。
  2. 上游 STORY-038-01/02 `verified`。
  3. `exporter.py` 文件所有权无冲突（S13 未并行写入）。
  4. CP5 全量确认通过 + 本 Story `dev_gate == cp5-approved`。
  5. 不修改已 approve 对象与 `topology_model.py`/`parser.py`（S13 primary）。

---

## 13. 风险与开放项

### 13.1 风险清单

| 风险 ID | 风险 | 影响 | 应对 |
|---------|------|------|------|
| RA-06-01 | `compute_ip_plan` 对含 SW 拓扑抛 `IP_PLAN_SW_VLAN_MISSING`，环回完整 ip_plan 无法端到端生成 | 环回 env-file 缺 IP 无法直接执行 | S06 单测用手构 `ip_plan` 隔离；完整 SW L2 域 IP 规划由后续 Story 或 vlan_assign 供给（见 §13.3） |
| RA-06-02 | `interface_kind` 语义边界（哪些接口标记 pppoe）由 S13 定义，S06 仅做通用跳过 | 若语义理解偏差，pppoe 接口范围误判 | §13.2 明确边界 + duck-typing 通用机制 + S13 LLD 回写 |
| RA-06-03 | 臂配对对复杂拓扑（多环/分支）不适用 | 非单一简单环被 §6.3 拦截，不会静默错配 | 闭环校验先行，非法拓扑显式报错，不穷举（RA-038-004 首版） |
| RA-06-04 | `_build_links` 折叠臂 `name` 语义与单链路 `name=lid` 不同 | 下游若依赖 link name 需确认 | 折叠臂 name=port_key，case_runner 仅断言非空，风险低；§13.2 记录 |

### 13.2 实现灰区与取舍记录

| 灰区 | 选项 | 决策 | 影响面 | 证据 | 重访条件 |
|------|------|------|--------|------|---------|
| `interface_kind` 读取方式（S13 加字段前） | A. duck-typing `getattr` 缺省 physical；B. S06 提前加字段（越权 S13） | **A** | exporter.py 内聚，S13 加字段后零改动 | ADR-CR038-05「InterfaceInfo 增 interface_kind（默认 physical）」归 S13 | S13 交付后确认字段名一致（`interface_kind`） |
| pppoe 标记粒度（DUT pppoe-facing 接口 vs SW pppoe-server 接口） | A. S06 通用跳过机制，不定义具体端口；B. S06 硬编码 DUT 侧 | **A** | S06 只做机制，具体端口语义由 S13 parser 注入 | Feature DESIGN §4「SW 端接口 interface_kind='pppoe'」与 HLD「dut1 接口读 interface_kind」存在表述差异 | S13 LLD 必须闭环此语义 |
| pppoe 臂 TG 侧是否豁免 IP | A. S06 仅 DUT 侧豁免，TG 侧仍 physical；B. 整臂豁免 | **A** | TG 接口 schema 无 interface_kind，S06 不改 TG | ADR-CR038-05 明确「pppoe 接口不产 ip」作用于 DUT | S13 若需整臂豁免，扩展 TG 侧 |
| 折叠臂 links `name` | A. name=port_key；B. name=直接 link id/拼接 | **A** | 折叠臂无单一 lid，port_key 最简洁 | HLD §10.3 未规定环回 links name | case_runner 若依赖 name 则回访 |
| N SW 上限 | A. 无硬上限；B. 设上限 | **A** | ADR-CR038-04「任意 N SW」 | 实际由闭环校验（度=2 单一环）约束 | 多环/分支需求出现时 CR 扩展 |

- 本 Story **无 `blocks_lld=true` 的 clarification item**；上述灰区均为 agent-default 决策或移交 S13 的非阻断 OPEN。唯一非阻断 ledger 项（`LCQ-STORY-038-06-01`）记录 pppoe 标记粒度边界，供 S13 LLD 消费。

### 13.3 OPEN / Spike

| 编号 | 状态 | 内容 | 归属 |
|------|------|------|------|
| OPEN-06-01 | 非阻断 OPEN | pppoe 标记粒度（DUT pppoe-facing vs SW pppoe-server）最终语义 | STORY-038-13（merge_owner）闭环 |
| OPEN-06-02 | 非阻断 OPEN | 环回完整 ip_plan 端到端生成（需 vlan_assign 或后续 IPAM 扩展） | 后续 Story / CR |

---

## 14. 验收与交接

### 14.1 Definition of Done

1. `_validate_mvp_topology` 满足 §3.1 放行规则（1 TG + 1 DUT + N SW，拒绝 Mock/PC/多 DUT/多 TG/UNKNOWN/MULTI）。
2. 环回 env-file `nodes` 含 `swN`（host/brand/role/interfaces），`port_mapping` 不含 SW 键，不新增 `${ENV.sw.*}`。
3. links 去重/自环/闭环校验按 §5.3 确定性算法生效，错误码 `LOOPBACK_SELF_LOOP`/`LOOPBACK_CLOSED_LOOP` 结构化暴露。
4. `interface_kind` 透传起点按 §3.5/§3.6 生效，pppoe 接口跳过 IP 填充与完备性校验（§3.7）。
5. 全部单测（T-06-01~T-06-14）通过，既有 `test_exporter.py` 回归全绿。
6. CP6 编码完成检查写入 `process/checks/CP6-STORY-038-06-exporter-loopback-mvp-CODING-DONE.md`。

### 14.2 交接摘要（供 meta-qa）

- 验证入口：`uv run --python 3.11 pytest skills/topo-planning/tests/`。
- 关键场景：SCN-038-POS-006（环回三层导出）、SCN-038-BND-004（Mock/PC/多 DUT 拒绝）、SCN-038-NEG-005（自环/闭环结构化错误）。
- 风险提示：`compute_ip_plan` 对 SW 拓扑的 `IP_PLAN_SW_VLAN_MISSING` 限制（S06 单测用手构 ip_plan 隔离）；`interface_kind` 语义边界待 S13 闭环（OPEN-06-01）。
- 设计 delta：本 Story 无长期 Feature DESIGN / ADR / HLD 回写（无 design-delta）。
