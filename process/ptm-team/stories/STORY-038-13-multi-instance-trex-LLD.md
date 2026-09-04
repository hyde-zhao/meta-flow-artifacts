---
doc_type: LLD
story_id: STORY-038-13
story_slug: multi-instance-trex
cr_id: CR-038
feature_id: F-CR038-E3
tier: A
required_level: full-lld
design_evidence_type: full-lld
lld_policy_required_level: full-lld
status: lld-ready-for-review
created_at: "2026-08-15"
owner: meta-dev
source_feature_design: docs/features/cr038-loopback-envfile/DESIGN.md
source_hld: docs/design/HLD-CR-038.md §10.2 / §10.3 / §10.5
source_adr: docs/design/ARCHITECTURE-DECISION-CR-038.md ADR-CR038-02 / ADR-CR038-05
depends_on:
  - STORY-038-02
  - STORY-038-03
  - STORY-038-06
merge_owner: STORY-038-13
---

# STORY-038-13 多实例 TRex + interface_kind 透传 — full-lld

> Tier-A：data-model（`InterfaceInfo.interface_kind` / `NodeInfo.trex_instance` / `trex_instances` 块）+ cross-module-contract（physical_pool → pool_merge → exporter 三层透传契约）+ shared-story-boundary（physical_pool.py/pool_merge.py/exporter.py 三共享文件，本 Story 为 merge_owner）三触发叠加，影响 5 个源码文件。

---

## 0. 工程依据

本 Story 是 F-CR038-E3（环回 env-file）的跨模块收口（merge_owner），落地多实例 TRex 归并与 interface_kind 全链路透传，跨 5 个源码文件。

- **HLD §10.2/§10.3/§10.5**：多实例 TRex 归并（`trex_instances` 汇总块）+ interface_kind 透传链路（topology_model → parser → exporter → _resolve_ip_plan）。
- **ADR-CR038-02**：多实例 TRex 归并（physical_pool `trex_instances` 为真相源，`_merge_tg` 按 name 合并）；**ADR-CR038-05**：interface_kind 透传（pppoe 接口不产 ip）。
- **Feature DESIGN（cr038-loopback-envfile）**：多实例 schema + 透传契约。
- **REQUIREMENTS**：R-F-013~017（多实例 TRex + interface_kind 透传）。

**文档结构导览（14 段语义要点对齐）**：§1 目标与需求，§2 文件影响范围（代码结构，模块拆分见 §2），§3 接口设计（API），§4 数据模型，§5 核心流程与技术细节，§6 异常处理，§7 测试设计，§8 实施步骤，§9 回滚策略，§10 安全约束，§11 需求契约映射，§12 依赖与前置，§13 风险与开放项，§14 验收与交接（DoD）。

---

## 1. 背景与目标

### 1.1 背景

CR-037 交付了「单 TG + 单 DUT 直连」env-file 导出，其 TG 侧数据源为 `physical_pool.yaml` 中 TG 节点 `management.api_server` 单值。CR-038 参考场景（`TG1 TE 实例 → SW2 → DUT2 → SW3(pppoe-server) → SW1 → TG1 TE2_1`）要求 TG 主机承载 **GE/TE 多个 TRex 实例**，且 DUT 侧接口需要区分 `interface_kind`（physical / pppoe），以支撑 PPPoE 环回拨号场景。

现状缺口：
- `physical_pool.py` 只有单值 `management.api_server` 与 `interfaces[].trex_port`，无 `trex_instances` 汇总块与接口级 `instance` 归属。
- `pool_merge.py` 的 `_merge_tg` 只归并单值 `api_server`/`sub_type`/`hardware_platform`，无多实例归并与 `${TG_*}` 占位符解析。
- `exporter.py` 的 `_build_nodes` 硬编码 `tg1` 节点仅产出 `trex_api_url`；`_build_nodes` 中 dut 接口硬编码 `interface_kind: 'physical'`。
- `topology_model.py` / `parser.py` 无 `interface_kind` / `trex_instance` 字段，逻辑拓扑无法声明 PPPoE 接口与目标实例。

### 1.2 目标（量化）

| 编号 | 目标 | 度量 |
|---|---|---|
| G1 | `physical_pool.py` 汇总 `trex_instances` 块为权威来源 | `get_trex_instances(device_name)` 返回 `{name: {api_server, hardware_platform, sync_port, async_port}}`，键集合 == 物理池 `trex_instances` 块键集合 |
| G2 | 接口级 `instance` 归属可反查 | `get_trex_instance_of_port(device_name, port_id)` 对含 `instance` 的端口返回 `(instance_name, instance_block)`；无 `instance` 返回 `(None, None)` |
| G3 | `get_free_ports` 透传实例信息 | 返回 dict 每项新增 `instance` + `trex_port` 两字段，且 `instance` 缺省为 `None` |
| G4 | pool_merge 按 name 归并 | `_merge_tg` 产出 `UnifiedNode.trex_instances` 键 == pool 侧 `trex_instances` 块键；devices 侧 `instances[name]` 仅补 pool 侧缺省字段 |
| G5 | exporter 多实例导出 | 有 `trex_instances` 块或声明 `trex_instance` 时 tg1 节点产出 `trex_instance`/`trex_sync_port`/`trex_async_port` + `trex_api_url` |
| G6 | 单实例回退 | 无 `trex_instances` 块且未声明 `trex_instance` 时 tg1 节点仅产出 `trex_api_url`（现状不变，R-NF-004） |
| G7 | interface_kind 透传 | dut1 接口 `interface_kind` == 逻辑 dut 节点对应接口 `interface_kind`（缺省 `'physical'`）；`interface_kind == 'pppoe'` 的接口不产 `ip`、`next_hop` 保持 `None` |
| G8 | 回归无破坏 | `node2_dut1_tg1_linkN` 系列 env-file 的 tg1/dut1 键集与原契约一致（dut 接口 `interface_kind` 恒为 `physical`） |

### 1.3 非目标

- 不改变 ADR-09 的 9 类 `${ENV.*}` 占位符（`tg.port1/port1.ip/port1.gw/url`、`dut.port1/port1.ip/url/next_hop`、`tg.ports[]`）。
- 不产生 `tg2` 多节点键名（本 Story 单逻辑 TG 节点；`tg2.*` 为未来多逻辑 TG 节点命名预留，见 LCQ-STORY-038-13-03）。
- 不改 `topo_mapper.py`、`switch_configurator.py`、`commands.py`、`case_runner.py`。
- 不实现真机下发（独立 runtime_authorization）。

---

## 2. 文件影响范围

| 文件 | 所有权 | 变更类型 | 变更内容（本 Story 增量） |
|---|---|---|---|
| `skills/topo-config/src/topology_model.py` | primary | 修改 | `InterfaceInfo` 新增 `interface_kind`（默认 `'physical'`）；`NodeInfo` 新增 `trex_instance`（默认 `None`） |
| `skills/topo-config/src/parser.py` | primary | 修改 | `_parse_interface` 解析 `interface_kind`；`_parse_yaml_topology` 节点构造解析 `trex_instance` |
| `skills/topo-config/src/physical_pool.py` | shared（S03） | 修改（merge_owner） | 新增 `get_trex_instances` / `get_trex_instance_of_port`；`get_free_ports` 透传 `instance` + `trex_port` |
| `skills/topo-planning/src/pool_merge.py` | shared（S02） | 修改（merge_owner） | `UnifiedNode.trex_instances` + `UnifiedInterface.instance`；`_merge_tg` 按 name 归并；`to_pool_node` 序列化 `trex_instances`/`instance`；新增 `_resolve_env` |
| `skills/topo-planning/src/exporter.py` | shared（S06） | 修改（merge_owner） | 新增 `_resolve_tg_instance`；`_build_nodes` 产 `trex_instance`/`trex_sync_port`/`trex_async_port` + 透传 `interface_kind`；`_resolve_ip_plan` 对 pppoe 接口跳过 ip 填充与完备性校验 |
| `skills/topo-config/tests/`、`skills/topo-planning/tests/` | primary | 新增 | 各模块单测 + fixture（TASK-E3-13-06） |

**增量合并边界（merge_owner 职责，不得覆盖前序改动）**：
- `physical_pool.py`：S03 已新增 `get_pppoe_server` / `validate_pppoe_server` / `_validate_password_cipher`（S03 LLD §3.3），本 Story 只**追加** 3 个 TG 多实例接口与 `get_free_ports` 的 2 个透传字段，不触碰 pppoe 相关代码。
- `pool_merge.py`：S02 已写 SW 归并（`_merge_sw_pc` 补 switch 块字段 + `_resolve_ledger` 覆盖 SW），本 Story 只改 `_merge_tg`、`_build_interfaces`、`UnifiedNode`/`UnifiedInterface` 数据类与 `to_pool_node`，不触碰 `_merge_sw_pc`/`_merge_dut`/`_resolve_ledger` 既有逻辑。
- `exporter.py`：S06 已写 `_validate_mvp_topology` 放宽 + `_build_links` 去重/自环/闭环 + `_build_nodes` SW 节点 + interface_kind 消费起点，本 Story 只改 `_build_nodes` 的 tg1 节点字段与 dut 接口 `interface_kind` 取值来源、`_resolve_ip_plan` 的 pppoe 跳过条件、新增 `_resolve_tg_instance`，不触碰 `_validate_mvp_topology`/`_build_links`/`_build_port_mapping`。

---

## 3. 接口设计

### 3.1 physical_pool.py 新增接口

```python
def get_trex_instances(self, device_name: str) -> dict:
    """读取 TG 节点 trex_instances 汇总块（ADR-CR038-02 唯一真相源 = physical_pool）。
    返回 {instance_name: {api_server, hardware_platform, sync_port, async_port}}；
    节点不存在 / 节点无 trex_instances 块 → 返回 {}。
    只读不写，不做校验。
    """
    node = self._device_by_name.get(device_name)
    if not node:
        return {}
    block = node.get('trex_instances') or {}
    return block if isinstance(block, dict) else {}


def get_trex_instance_of_port(self, device_name: str, port_id: str):
    """按接口级 instance 反查实例块。
    返回 (instance_name, instance_block)；端口无 instance 字段 / 实例名不存在 → (None, None)。
    """
    node = self._device_by_name.get(device_name)
    if not node:
        return (None, None)
    iface = (node.get('interfaces') or {}).get(port_id)
    if not isinstance(iface, dict):
        return (None, None)
    inst_name = iface.get('instance')
    if not inst_name:
        return (None, None)
    instances = self.get_trex_instances(device_name)
    block = instances.get(inst_name)
    if block is None:
        return (None, None)
    return (inst_name, block)
```

`get_free_ports` 修改：返回 dict 每项在既有 `port/physical_port/speed_class/media_type` 基础上**新增** `instance`（`iface.get('instance')`，缺省 `None`）与 `trex_port`（`iface.get('trex_port')`，缺省 `None`）。判定逻辑（`allocated_to is None` + speed/media 过滤）不变。

### 3.2 pool_merge.py 接口

```python
def _resolve_env(value):
    """解析 ${TG_HARDWARE_PLATFORM} / ${TG_API_SERVER} 等环境变量占位符。
    命中 ^\$\{([A-Z_][A-Z0-9_]*)\}$ 且 os.environ 含该变量 → 返回实际值；
    命中但环境变量不存在 → 保留原串 + logger.warning（不抛错、不阻断）；
    非占位符格式 → 原样返回。
    """
```

`_merge_tg` 修改：在既有单值归并（`api_server`/`sub_type`/`hardware_platform`）基础上**追加**：
1. `node.trex_instances = {name: _resolve_instances_block(block)}`（pool 侧 `trex_instances` 块为真相源，逐字段经 `_resolve_env` 解析）。
2. devices 侧 `tg.instances[name]`（若存在）仅补 pool 侧该实例缺省的 `sync_port`/`async_port`/`port_map`/`ports`/`service`（pool 侧已有字段不被覆盖）。
3. `_build_interfaces` 从 `interfaces[port].instance` 读入 `UnifiedInterface.instance`。

`UnifiedNode.to_pool_node` 修改：节点 dict 新增 `trex_instances`（`self.trex_instances`，非空时写入）；interfaces 序列化新增 `instance`（`iface.instance`，非 `None` 时写入）。

### 3.3 exporter.py 接口

```python
def _resolve_tg_instance(self, tg_phys: str, logical_tg) -> tuple:
    """推导逻辑 TG 节点使用的 TRex 实例。
    返回 (instance_name, instance_block)。确定性规则：
      1. logical_tg.trex_instance 声明且实例存在 → (name, block)。
      2. 声明但实例不存在 → raise ExporterError('TREX_INSTANCE_NOT_FOUND', ...)。
      3. 未声明且 len(trex_instances) == 1 → 取唯一实例 (name, block)。
      4. 未声明且 len(trex_instances) > 1 → raise ExporterError('TREX_INSTANCE_AMBIGUOUS', ...)。
      5. 未声明且 trex_instances 为空（node2 系列）→ (None, {'api_server': management.api_server,
         'sync_port': None, 'async_port': None})，向后兼容。
    """
```

`_build_nodes` 修改：tg1 节点在保留 `trex_api_url` 基础上，当 `_resolve_tg_instance` 返回 `instance_name` 非 `None` 时，**新增** `trex_instance`（= instance_name）、`trex_sync_port`（= block.sync_port）、`trex_async_port`（= block.async_port）；`trex_api_url` 取值优先级 = `block.api_server`（非空时）→ `self._tg_api_server(tg_phys)` 回退。

### 3.4 topology_model.py / parser.py 接口

```python
# topology_model.py
@dataclass
class InterfaceInfo:
    ...
    interface_kind: Optional[str] = 'physical'   # 新增：physical | pppoe（默认 physical）

@dataclass
class NodeInfo:
    ...
    trex_instance: Optional[str] = None          # 新增：TG 节点目标 TRex 实例名（GE/TE）
```

```python
# parser.py _parse_interface 追加
interface_kind=iface_raw.get('interface_kind', 'physical'),
# parser.py 节点构造追加
trex_instance=n.get('trex_instance'),
```

---

## 4. 数据模型

### 4.1 physical_pool.yaml TG 节点新增（模板，占位/勿直接使用）

```yaml
  TG1:
    node_type: TG
    device_id: ${DEVICE_ID_TG1}
    device_type: ${DEVICE_TYPE_TG}
    hardware_platform: ${HARDWARE_PLATFORM_TG}
    management:
      api_server: "${API_SERVER_TG1}"          # 单实例回退值（node2 系列兼容）
    # 多实例 TRex 汇总块（ADR-CR038-02 唯一真相源），键 = 实例名（GE/TE）
    trex_instances:
      GE:
        api_server: "${TG_API_SERVER_GE}"        # 可为占位符，_resolve_env 解析
        hardware_platform: "${TG_HARDWARE_PLATFORM_GE}"
        sync_port: "${TG_SYNC_PORT_GE}"          # trex 同步端口
        async_port: "${TG_ASYNC_PORT_GE}"        # trex 异步端口
      TE:
        api_server: "${TG_API_SERVER_TE}"
        hardware_platform: "${TG_HARDWARE_PLATFORM_TE}"
        sync_port: "${TG_SYNC_PORT_TE}"
        async_port: "${TG_ASYNC_PORT_TE}"
    interfaces:
      port1:
        physical_port: ${PHYSICAL_PORT_TG1_1}
        speed_class: ${SPEED_CLASS_TG1_1}
        media_type: ${MEDIA_TYPE_TG1_1}
        trex_port: "${TREX_PORT_TG1_1}"          # 如 "2_3"
        instance: TE                              # 接口级实例归属（可选，缺省 None）
```

### 4.2 devices.yaml tg 块多实例补充（可选，LCQ-STORY-038-13-01 默认方案 A）

```yaml
    tg:
      host: 10.113.52.253
      sub_type: trex
      hardware_platform: C236
      api_server: 10.113.52.253:8450             # 单实例回退
      instances:                                 # 可选补充块
        TE:
          port_map: {"2_3": "1/1/1"}             # trex_port -> physical_port
          ports: ["1/1/1", "1/1/2"]
          service: {sync_port: "0/0/2", async_port: "0/0/3"}
```

### 4.3 归并数据类

```python
@dataclass
class UnifiedNode:
    ...
    trex_instances: Dict[str, Dict] = field(default_factory=dict)   # 新增：{name: {api_server, hardware_platform, sync_port, async_port, port_map?, ports?, service?}}

@dataclass
class UnifiedInterface:
    ...
    instance: Optional[str] = None                # 新增：TG 接口实例归属（GE/TE）
```

### 4.4 env-file 产物 schema（tg1/dut1 节点）

```yaml
nodes:
  tg1:                                            # 多实例/声明场景
    trex_api_url: "http://10.113.52.253:8450"
    trex_instance: "TE"                           # 新增（有 trex_instances 或声明时）
    trex_sync_port: "0/0/2"                       # 新增（可空 None）
    trex_async_port: "0/0/3"                      # 新增（可空 None）
    interfaces:
      port1: {ip: "198.18.0.2/24", gw: "198.18.0.1"}
  dut1:
    host: "https://10.113.55.51"
    next_hop: null                                # interface_kind=pppoe 时保持 None
    interfaces:
      port1: {id: "if-001", interface_kind: "pppoe"}   # 透传，pppoe 无 ip
      port2: {id: "if-002", interface_kind: "physical", ip: "198.18.1.1/24"}
```

**node2 系列回退（无 trex_instances、无声明）**：tg1 节点保持现状，仅 `trex_api_url` + `interfaces`，**不新增** `trex_instance`/`trex_sync_port`/`trex_async_port`；dut 接口 `interface_kind` 恒 `'physical'`（回归无破坏）。

### 4.5 持久化

无新增持久化。`trex_instances` / `interfaces[].instance` 随 `physical_pool.yaml` 静态存在；`UnifiedNode.trex_instances` 经 `to_pool_data()` → `to_pool_node()` 序列化回 `physical_pool.yaml` schema，供 `PhysicalPool.from_pool_data()` 消费（`get_trex_instances` 只读）。

---

## 5. 核心流程

### 5.1 多实例 TRex 透传（R-F-013/014/015）

```
physical_pool.yaml trex_instances + interfaces[].instance
  → PhysicalPool.get_trex_instances / get_trex_instance_of_port（§3.1，只读）
  → pool_merge._merge_tg（按 name 归并：pool 侧为准 + _resolve_env 占位解析 + devices 侧补缺省）
  → UnifiedPool.to_pool_data() → to_pool_node()（序列化 trex_instances + instance）
  → PhysicalPool.from_pool_data(final_pool, ledger_path)
  → exporter._resolve_tg_instance(tg_phys, logical_tg)（声明→按名取；单实例→唯一；多实例未声明→报错；无块→回退 api_server）
  → env-file nodes.tg1: trex_instance / trex_sync_port / trex_async_port / trex_api_url
```

### 5.2 interface_kind 透传（R-F-016/017）

```
逻辑拓扑 dut 节点 interfaces[port].interface_kind（parser 解析，缺省 physical）
  → exporter._build_nodes：port_key → 逻辑 dut 接口反查（via_link → PortMapping.logical_port）
  → dut1.interfaces[port_key].interface_kind = 逻辑接口 interface_kind
  → _resolve_ip_plan：interface_kind == 'pppoe' 的接口跳过 ip 填充 + 完备性校验；
     next_hop 仅对 physical 接口域填充，pppoe 域不参与，next_hop 保持 None
```

### 5.3 `_resolve_tg_instance` 决策表（确定性）

| 条件 | 行为 | 结果 |
|---|---|---|
| `trex_instance` 声明 + 实例存在于 `trex_instances` | 按名取 | `(name, block)` |
| `trex_instance` 声明 + 实例不存在 | 抛错 | `TREX_INSTANCE_NOT_FOUND` |
| 未声明 + `len(trex_instances) == 1` | 取唯一实例 | `(name, block)` |
| 未声明 + `len(trex_instances) > 1` | 抛错 | `TREX_INSTANCE_AMBIGUOUS` |
| 未声明 + `trex_instances` 为空 | 回退单值 | `(None, {api_server: management.api_server, sync_port: None, async_port: None})` |

---

## 6. 异常处理

| 异常场景 | 触发位置 | 行为 | 错误码 |
|---|---|---|---|
| 逻辑声明 `trex_instance` 但 pool 无该实例 | `exporter._resolve_tg_instance` | 抛结构化错误，终止导出 | `TREX_INSTANCE_NOT_FOUND` |
| `trex_instances` 块 `len>1` 且未声明 `trex_instance` | `exporter._resolve_tg_instance` | 抛结构化错误，终止导出 | `TREX_INSTANCE_AMBIGUOUS` |
| `trex_instances` 块非 dict（YAML 误写为 list） | `physical_pool.get_trex_instances` | 返回 `{}`（视为无块，回退单值） | 无（降级） |
| 端口 `instance` 指向不存在的实例名 | `physical_pool.get_trex_instance_of_port` | 返回 `(None, None)` | 无（降级） |
| `${TG_*}` 占位符无对应环境变量 | `pool_merge._resolve_env` | 保留原串 + `logger.warning`，不阻断 | 无（降级） |
| dut 接口 `interface_kind` 非法（非 physical/pppoe） | `exporter._resolve_ip_plan` | 按非 pppoe 处理（仍校验 ip 完备性），不静默跳过 | 无（保守） |
| 多实例块缺 `sync_port`/`async_port` | `_merge_tg` / `_resolve_tg_instance` | 字段置 `None`，devices 侧补缺省，仍产出 | 无（可选字段） |

**失败路径原则**：多实例歧义与实例不存在属**配置缺失**，必须显式阻断（不静默选第一个、不回退单值）；占位符与可选字段缺失属**可降级**，保留原串/置 None 并告警。

---

## 7. 测试设计

### 7.1 单元测试清单

| 测试用例 | 模块 | 断言 |
|---|---|---|
| `test_get_trex_instances_present` | physical_pool | TG 节点含 `trex_instances` 块 → 返回 `{GE:{...}, TE:{...}}`，键与块一致 |
| `test_get_trex_instances_absent` | physical_pool | 无块 / 节点不存在 → `{}` |
| `test_get_trex_instance_of_port_hit` | physical_pool | `interfaces[port].instance=TE` → `('TE', block)` |
| `test_get_trex_instance_of_port_miss` | physical_pool | 无 instance / 实例不存在 → `(None, None)` |
| `test_get_free_ports_passthrough` | physical_pool | 空闲端口 dict 含 `instance` + `trex_port`（缺省 `None`） |
| `test_merge_tg_trex_instances` | pool_merge | `UnifiedNode.trex_instances` 键 == pool 侧块键；devices 侧补缺省字段 |
| `test_merge_tg_instances_pool_priority` | pool_merge | pool 侧已有 `sync_port` 不被 devices 侧覆盖（pool 侧为准） |
| `test_resolve_env_resolved` | pool_merge | `${TG_API_SERVER}` 命中环境变量 → 实际值 |
| `test_resolve_env_unresolved` | pool_merge | 环境变量缺失 → 保留原串 + warning |
| `test_to_pool_node_serializes_instances` | pool_merge | `to_pool_data()` 产物节点含 `trex_instances` + `interfaces[port].instance` |
| `test_resolve_tg_instance_declared` | exporter | 声明 TE + 块含 TE → `('TE', block)` |
| `test_resolve_tg_instance_single_fallback` | exporter | 未声明 + 单实例块 → 唯一实例 |
| `test_resolve_tg_instance_ambiguous` | exporter | 未声明 + 多实例 → `TREX_INSTANCE_AMBIGUOUS` |
| `test_resolve_tg_instance_not_found` | exporter | 声明不存在实例 → `TREX_INSTANCE_NOT_FOUND` |
| `test_resolve_tg_instance_node2_fallback` | exporter | 无块 → `(None, {api_server, sync_port:None, async_port:None})` |
| `test_build_nodes_multi_instance`（SCN-038-POS-010） | exporter | tg1 节点含 `trex_instance`/`trex_sync_port`/`trex_async_port`/`trex_api_url` |
| `test_build_nodes_node2_no_new_keys`（SCN-038-BND-003） | exporter | tg1 节点仅 `trex_api_url`，无新增三字段 |
| `test_interface_kind_pppoe_no_ip`（SCN-038-POS-011） | exporter | `interface_kind=pppoe` 接口无 `ip`、`next_hop=None` |
| `test_interface_kind_physical_ip_checked` | exporter | physical 接口仍校验 ip 完备性（缺 ip → `IP_PLAN_INCOMPLETE`） |
| `test_parser_interface_kind` | parser | `_parse_interface` 读 `interface_kind`（缺省 `physical`） |
| `test_parser_trex_instance` | parser | 节点构造读 `trex_instance` |

### 7.2 测试数据 fixture

- `skills/topo-config/tests/fixtures/pool_trex_multi_instance.yaml`：TG 节点含 `trex_instances`（GE/TE 两块）+ `interfaces[port].instance`。
- `skills/topo-planning/tests/fixtures/pool_merge/devices_trex_multi.yaml`：devices.yaml tg 块含 `instances` 补充块（用于 `_merge_tg` 补缺省测试）。
- 逻辑拓扑 fixture：dut 节点接口 `interface_kind: pppoe` + tg 节点 `trex_instance: TE`（复用 S06 环回拓扑 fixture 扩展）。

### 7.3 验证入口

```
uv run --python 3.11 pytest skills/topo-config/tests/ skills/topo-planning/tests/
```

---

## 8. 实施步骤

按依赖序（数据模型 → 数据源 → 归并 → 导出 → 单测）：

| 步骤 | TASK-ID | 动作 | 输出 |
|---|---|---|---|
| 1 | TASK-E3-13-04 | `topology_model.py` `InterfaceInfo.interface_kind`（默认 `'physical'`）+ `NodeInfo.trex_instance`（默认 `None`） | topology_model.py |
| 2 | TASK-E3-13-05 | `parser.py` `_parse_interface` 解析 `interface_kind`、节点构造解析 `trex_instance` | parser.py |
| 3 | TASK-E3-13-01 | `physical_pool.py` 新增 `get_trex_instances` / `get_trex_instance_of_port`；`get_free_ports` 透传 `instance`+`trex_port` | physical_pool.py |
| 4 | TASK-E3-13-02 | `pool_merge.py` `UnifiedNode.trex_instances` + `UnifiedInterface.instance`；`_merge_tg` 按 name 归并 + `_resolve_env`；`to_pool_node` 序列化 | pool_merge.py |
| 5 | TASK-E3-13-03 | `exporter.py` 新增 `_resolve_tg_instance`；`_build_nodes` 产多实例字段 + 透传 `interface_kind`；`_resolve_ip_plan` pppoe 跳过 | exporter.py |
| 6 | TASK-E3-13-06 | 单测 + fixture（POS-010/011、BND-003） | tests/ |

每步完成后跑该模块单测；步骤 6 全量回归。

---

## 9. 回滚策略

- **逐文件回退**：5 个源码文件 + 测试均增量追加/修改，任一文件回退不影响其余；回退顺序与实施顺序相反（exporter → pool_merge → physical_pool → parser → topology_model）。
- **数据模型向后兼容**：`interface_kind` 默认 `'physical'`、`trex_instance` 默认 `None`、`trex_instances` 缺省 `{}`，回退后 CR-037 逻辑拓扑（无新字段）解析结果不变。
- **env-file 回退**：无 `trex_instances` 块/无声明时 tg1 节点不新增三字段，产物与 CR-037 一致；`interface_kind` 缺省 `physical` 与原硬编码值等价。
- **无台账副作用**：本 Story 不写 `physical_ledger.yaml`、不调用 `allocate`/`release`，无占用状态回退负担。
- **切换条件（ADR-CR038-02）**：若实例改由 devices.yaml 唯一定义，调整 `_merge_tg` 合并方向（pool 侧为准 → devices 侧为准），回退点为 §3.2。

---

## 10. 安全约束

- **GE1_1~4 实例禁改动**：本 Story 只操作 TE 实例与 TE2_1~4 端口；`get_trex_instances` 只读、不写台账、不触发下发。
- **凭据占位**：`trex_instances` 块不含密码字段；若未来含凭据必须 `${ENV_VAR}` 占位（ADR-02）。`_resolve_env` 只解析非凭据的 `${TG_*}` 字段，不解析 `${PASSWORD*}`/`${*_PASSWORD}`/token/secret。
- **dry-run 默认**：本 Story 仅数据模型 + 归并 + env-file 导出，不真机下发（独立 runtime_authorization）。
- **禁止明文**：新增字段均非凭据；`_resolve_env` 不触碰 `management.password`/`username`。
- **确定性**：`_resolve_tg_instance` 决策表无随机性；`_resolve_env` 同输入同输出；无时间戳注入（`generated_at` 沿用 S05 既有处理）。

---

## 11. 契约映射

| 需求/ADR | 落地 | 验证入口 |
|---|---|---|
| R-F-013（trex_instances 汇总） | physical_pool `get_trex_instances` | `test_get_trex_instances_*` |
| R-F-014（pool 侧为准按 name 合并） | pool_merge `_merge_tg` + `_resolve_env` | `test_merge_tg_trex_instances` / `test_merge_tg_instances_pool_priority` |
| R-F-015（多实例导出 + 单值回退） | exporter `_resolve_tg_instance` + `_build_nodes` | `test_resolve_tg_instance_*` / `test_build_nodes_*` |
| R-F-016（interface_kind 透传） | topology_model/parser + exporter `_build_nodes` | `test_parser_interface_kind` / `test_interface_kind_*` |
| R-F-017（pppoe 无 ip、next_hop=None） | exporter `_resolve_ip_plan` | `test_interface_kind_pppoe_no_ip` |
| ADR-CR038-02 | physical_pool `trex_instances` 唯一真相源 + `_merge_tg` 按 name 合并 | §3.1 / §5.1 |
| ADR-CR038-05 | `interface_kind` 默认 physical + pppoe 不产 ip | §4.4 / §5.2 |
| ADR-09（9 占位不变） | tg1/dut1 主实例键名不变，仅扩展 nodes 内字段 | SCN-038-POS-010/011、BND-003 |
| R-NF-004（回归无破坏） | 无块/无声明回退单值；interface_kind 缺省 physical | `test_build_nodes_node2_no_new_keys` |

**TASK-ID ↔ 文件影响一一对应**：TASK-E3-13-01→physical_pool.py；TASK-E3-13-02→pool_merge.py；TASK-E3-13-03→exporter.py；TASK-E3-13-04→topology_model.py；TASK-E3-13-05→parser.py；TASK-E3-13-06→tests/。

---

## 12. 依赖与前置

| 依赖 | 类型 | 门控 | 本 Story 消费 |
|---|---|---|---|
| STORY-038-02 | hard | `verified` | pool_merge.py 的 SW 归并基线（本 Story 在其上续写 `_merge_tg` 多实例） |
| STORY-038-03 | hard（file-conflict，physical_pool.py 串行） | `verified` | physical_pool.py 的 `get_pppoe_server`/`validate_pppoe_server` 基线（本 Story 追加 TG 多实例接口） |
| STORY-038-06 | hard（file-conflict，exporter.py 串行） | `verified` | exporter.py 的 MVP 放宽 + links 校验 + interface_kind 消费起点（本 Story 续写多实例导出 + 透传来源） |

前置确认：
- CP4 已 PASS（Story DAG 并行安全，本 Story 为 merge_owner，三共享文件按 Wave 串行）。
- CP5 全量设计证据人工确认通过 + 本 Story `dev_gate=cp5-approved` 满足后才进入实现。
- 澄清项 LCQ-STORY-038-13-01/02/03（`blocks_lld=false`）由 host-orchestrator 在 CP5 前批量询问并回填；不阻断本 LLD 落盘。

---

## 13. 风险与开放项

### 13.1 风险

| 风险 | 影响 | 应对 |
|---|---|---|
| S06 LLD 未产出，interface_kind 消费起点实现细节未知 | exporter.py 增量合并可能重复实现 | 本 LLD 定义 S13 最终行为（kind 取值来源 = 逻辑拓扑），S13 只改来源不重设计消费；若 S06 已硬编码则 S13 替换为透传 |
| devices.yaml 多实例补充 schema 未固化 | `_merge_tg` 补全逻辑歧义 | LCQ-STORY-038-13-01（默认方案 A 嵌套 instances） |
| 多实例未声明实例导致发流打错 | 验证失败 | LCQ-STORY-038-13-02（默认报错 TREX_INSTANCE_AMBIGUOUS） |
| `tg2.*` 键名验收口径歧义 | CP7 验收争议 | LCQ-STORY-038-13-03（默认仅 tg1 节点级） |

### 13.2 实现灰区与取舍记录

| 灰区 | 决策 | 证据 | 重访条件 |
|---|---|---|---|
| devices.yaml `port_map/ports/service` schema | 嵌套 `instances[name]`（方案 A） | LCQ-STORY-038-13-01 | 若 devices.yaml 需扁平单实例字段则切 B |
| 多实例未声明失败行为 | 报错 `TREX_INSTANCE_AMBIGUOUS`（方案 A） | LCQ-STORY-038-13-02 | 若主实例名契约固化则切 C |
| tg2 多节点键名边界 | 本 Story 仅 tg1 节点级（方案 A） | LCQ-STORY-038-13-03 | 多 TG 逻辑节点需求出现时后续 CR 扩展 |

### 13.3 开放项

- 无 BLOCKING。3 个 LCQ 均为 `blocks_lld=false`，已给确定性默认方案；用户 approve 即接受推荐。

---

## 14. 验收与交接

### 14.1 Definition of Done

- [ ] `topology_model.py` `InterfaceInfo.interface_kind`（默认 `'physical'`）+ `NodeInfo.trex_instance`（默认 `None`）落地。
- [ ] `parser.py` 解析 `interface_kind` / `trex_instance`，缺省值正确。
- [ ] `physical_pool.py` `get_trex_instances` / `get_trex_instance_of_port` / `get_free_ports` 透传落地，不破坏 S03 pppoe 接口。
- [ ] `pool_merge.py` `_merge_tg` 按 name 归并 + `_resolve_env` + 序列化落地，不破坏 S02 SW 归并。
- [ ] `exporter.py` `_resolve_tg_instance` + `_build_nodes` 多实例字段 + `interface_kind` 透传 + `_resolve_ip_plan` pppoe 跳过落地，不破坏 S06 MVP/links。
- [ ] SCN-038-POS-010（多实例 env-file）、SCN-038-POS-011（pppoe 无 ip、next_hop=None）、SCN-038-BND-003（单实例回退）全 PASS。
- [ ] 回归：`node2_dut1_tg1_linkN` 系列 env-file 键集与 CR-037 一致。
- [ ] CP6 编码完成检查结果写入。

### 14.2 交接摘要（供 meta-qa）

- 验证入口：`uv run --python 3.11 pytest skills/topo-config/tests/ skills/topo-planning/tests/`。
- 关键验证对象：`_resolve_tg_instance` 决策表（§5.3）、`interface_kind` 透传链路（§5.2）、`_merge_tg` 归并优先级（pool 侧为准）。
- 风险提示：3 个 LCQ 待 host-orchestrator 回填；S06 interface_kind 消费起点需实现时核对增量不重复。
- 证据路径：`process/returns/STORY-038-13.return.json`、`process/evidence/STORY-038-13.index.json`。
