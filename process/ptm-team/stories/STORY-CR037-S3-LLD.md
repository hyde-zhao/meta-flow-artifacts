---
doc_type: LLD
story_id: "CR037-S3"
story_slug: "pool-merge"
cr_id: "CR-037"
tier: "A"
status: "draft"
created_at: "2026-08-05"
owner: "meta-dev"
source_hld: "process/changes/CR-037-HLD.md §4.3"
source_story: "process/stories/STORY-CR037-S3.md"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "skills/topo-planning/src/pool_merge.py 新建"
---

# STORY-CR037-S3-LLD：pool_merge.py 详细设计

> **Tier**: A（多数据源归并 + TG 归并规则 H2 + 冲突消歧 + 台账三角映射）  
> **Shared Fragments**: `skills/topo-config/config/physical_pool.yaml`（模板，P-1 改造后）、`skills/device-management/reference/device-reference.md`（平台映射，只读消费）  
> **Open Items**: 无

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-08-05 | meta-dev | 初稿（14 章节 full-lld，CP5 提交） |
| v1.1 | 2026-08-05 | meta-dev（CP5 NEEDS_REWORK 修订） | S3-A：`device_groups` 实为 list（devices.yaml.example），修正 §5.1/§6.5/§8.4/§13 的 `.items()`/`.values()` 为 list 遍历，消除「遍历不到任何组」缺陷；S3-B：统一 S3→S4 池对象契约——TopoMapper 输入 raw PhysicalPool，S3 产出最终池数据（`UnifiedPool.to_pool_data()` + `ledger_path`），S4/S7 经 `PhysicalPool.from_pool_data()` 构造；S3-C：TG 关联 api_server 主键 + host 兜底；S3-D：A1500-HU 等型号消歧统一在 S2 平台别名层，S3 不重复决策 |
| v1.2 | 2026-08-05 | meta-dev（CP5 第二轮复审 Minor-1 修订） | §13 灰区1 伪代码修正：内联循环「api_server + host 同判」改为**显式决策表（api_server 主键 → host 兜底 → unmatched）+ 两遍完整循环**（先遍历全部 candidates 找 api_server 主键，再遍历找 host 兜底），消除「首组 host 兜底误命中、遮蔽后续 api_server 主键候选」缺陷；§10 新增 `test_tg_host_fallback_decision_table` 单测覆盖决策表边界；实现语义与 §6.5 `_find_device_group_by_api_server` 对齐 |

---

## 0. 工程依据

本模块是 CR-037 topo-planning skill 的第三数据层组件。工程依据源自 HLD §4.3（pool_merge 模块设计），AG-3 归并策略（推荐 A：devices 优先 + 备选 B：池全量承载降级），H2（TG 归并规则），以及 CR-037 §前置数据任务 P-3（台账三角映射并入 S3）。顶层目标是将目标项目分散在 physical_pool.yaml、devices.yaml、physical_ledger.yaml 三个文件的物理资源信息归并为**最终池数据（UnifiedPool）**：S3 仅做「校验/覆盖 devices 元数据」+ 台账三角映射，产出最终池数据；S4/S7 用该数据构造 raw `PhysicalPool`（`PhysicalPool.from_pool_data`，TopoMapper 输入）。**UnifiedPool 不是 S4 topo_mapper 的直接输入**（S3-B，评审 D 组契约）。

---

## 1. 目标

新建 `skills/topo-planning/src/pool_merge.py`，将目标项目的 `topology/physical_pool.yaml`（物理接线/端口池）与 `topology/devices.yaml`（设备型号/平台/管理地址）归并为统一物理池视图（`UnifiedPool`），完成：
1. DUT firewall host 关联
2. TG api_server 关联（H2）
3. 台账三角映射（`台账.model ↔ 池.device_type ↔ device-reference 平台`）

归并后的 `UnifiedPool` 是**最终池数据**：S4 topo_mapper 的输入是 raw `PhysicalPool`（由 S4/S7 用 `UnifiedPool.to_pool_data()` + `ledger_path` 经 `PhysicalPool.from_pool_data()` 构造，S3-B）。UnifiedPool 提供每个节点的完整视图（接线 + 型号 + 平台 + 管理地址），用于 S7 展示、校验与 S4/S7 构造 PhysicalPool。

## 2. Requirements

### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | 加载 `topology/physical_pool.yaml` → `PoolData`（含 nodes/links/metadata） | P0 |
| FR2 | 加载 `topology/devices.yaml` → `DevicesData`（含 device_groups） | P0 |
| FR3 | 加载 `topology/physical_ledger.yaml` → `LedgerData`（含 台账.model → 池.device_type） | P0 |
| FR4 | DUT host 关联：`device_groups[].firewall.host ↔ pool.nodes[].management.host` | P0 |
| FR5 | TG api_server 关联：`device_groups[].tg.api_server ↔ pool.nodes[].management.api_server`（H2） | P0 |
| FR6 | 台账三角映射：`台账.model ↔ 池.device_type ↔ device-reference 平台` | P0 |
| FR7 | 型号冲突处理：同 host 且 pool.device_type ≠ devices.device_type → warning + 以 devices 为准 | P0 |
| FR8 | 关联失败不阻断（标记 `unmatched_nodes[]`） | P1 |
| FR9 | 池全量承载降级：若物理池节点无需 devices 补充 → 仅校验 devices 存在性（备选 B） | P1 |
| FR10 | TG 节点补充 `sub_type`（ixia-c/trex）+ `hardware_platform`（EP/C236/J1900） | P0 |
| FR11 | 归并后每个 UnifiedNode 含完整视图：host、device_type、hardware_platform、interfaces（物理端口 + speed_class/media_type）、ledger 占用状态 | P0 |
| FR12 | `UnifiedPool` 可导出 physical_pool.yaml schema 的最终池数据（`to_pool_data()`），供 S4/S7 构造 raw `PhysicalPool`（S3-B，评审 D 组契约） | P0 |

### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 加载失败（文件不存在/YAML 语法错误）返回清晰错误 |
| NFR2 | 归并不修改物理池或 devices 原始文件（只读加载） |
| NFR3 | 模块可独立测试，不依赖 S1/S2 |

## 3. 模块拆分与职责

| 函数 | 职责 |
|------|------|
| `merge_pool(pool_path, devices_path, ledger_path)` | 主入口：加载 3 文件 → 归并 → UnifiedPool |
| `load_pool(pool_path) → PoolData` | 加载物理接线池 YAML |
| `load_devices(devices_path) → DevicesData` | 加载设备清单 YAML |
| `load_ledger(ledger_path) → LedgerData` | 加载设备台账 YAML |
| `_merge_dut(key, pool_node, device_group) → UnifiedNode` | DUT 节点归并（host 关联 + 冲突处理，node_id 用 pool_node.get('node_id', key)，P1-5） |
| `_merge_tg(key, pool_node, device_group) → UnifiedNode` | TG 节点归并（api_server 归一化关联 + sub_type/hardware_platform 补充，P1-4/P1-5） |
| `_merge_sw_pc(pool_node, devices_data) → UnifiedNode` | SW/PC 节点归并（仅校验存在性） |
| `_resolve_ledger(node, pool_node, ledger_data, platform_map) → None` | 台账三角映射：填充 model/platform（等价类）到 UnifiedNode（P0-3/P1-3/P2-5） |
| `_resolve_conflict(pool_node, device_group) → List[str]` | 冲突检测：返回 warnings |

## 4. 代码结构与文件影响范围

### 新增文件

```
skills/topo-planning/src/pool_merge.py                # 核心归并模块（~350 行）
```

### 只读消费

```
topology/physical_pool.yaml（目标项目，P-1 模板化后的 schema）
topology/devices.yaml（目标项目，device_groups 为 list，S3-A）
topology/physical_ledger.yaml（目标项目，台账记录）
skills/device-management/reference/device-reference.md（平台映射参考，只读）
```

### 修改文件

无。全部为新增。

## 5. 数据模型与持久化设计

### 5.1 核心数据类

```python
@dataclass
class UnifiedPool:
    """归并后的最终物理资源池数据（S3 产出）

    S3-B（评审 D 组契约）：本对象**不是** S4 topo_mapper 的直接输入。
    S4/S7 用 `to_pool_data()` 得到 final pool_data（physical_pool.yaml schema），
    再经 `PhysicalPool.from_pool_data(pool_data, ledger_path)` 构造 raw PhysicalPool
    （TopoMapper 输入）。
    """
    pool_id: str
    metadata: Dict[str, Any]
    nodes: Dict[str, UnifiedNode]              # node_id → UnifiedNode
    links: List[Dict[str, Any]]                # 物理接线 links
    ledger_path: Optional[str] = None          # 台账路径（S4/S7 构造 PhysicalPool 用，S3-B）
    unmatched_nodes: List[str]                 # 关联失败的节点 ID
    merge_warnings: List[str]                  # 归并过程中的 warnings

    def to_pool_data(self) -> Dict[str, Any]:
        """导出为 physical_pool.yaml schema 的最终池数据 dict（供 PhysicalPool 构造，S3-B）"""
        nodes = {nid: node.to_pool_node() for nid, node in self.nodes.items()}
        return {'metadata': self.metadata, 'nodes': nodes, 'links': self.links}

@dataclass
class UnifiedNode:
    """归并后的单个物理节点"""
    node_id: str                               # pool 中的节点 key（如 FW1，P1-5：pool_node.get('node_id', key)）
    node_type: str                             # DUT | TG | SW | PC
    device_id: Optional[str] = None            # 台账/池 device_id（如 201，供台账关联与 S4 占用闭环，P0-3）
    device_type: Optional[str] = None          # TGFW 型号（如 DAS-TGFW-A1600-HU）
    hardware_platform: Optional[str] = None    # 硬件等价类（如 HG3250，P1-3）
    sub_type: Optional[str] = None             # TG 子类型（ixia-c/trex）
    management_host: Optional[str] = None
    management_port: Optional[int] = None
    api_server: Optional[str] = None           # TG 专用
    username: Optional[str] = None
    password: Optional[str] = None             # 占位符
    interfaces: Dict[str, UnifiedInterface] = field(default_factory=dict)
    ledger_model: Optional[str] = None         # 台账型号（如 DAS-TGFW-A1600-HU，与 device_type 同值直通，P2-5）
    ledger_status: Optional[str] = None        # 台账占用状态
    source: str = "pool"                       # "pool" | "devices" | "merged"

    def to_pool_node(self) -> Dict[str, Any]:
        """还原为 physical_pool.yaml 节点 dict（供 UnifiedPool.to_pool_data()，S3-B）"""
        node = {
            'node_type': self.node_type,
            'device_id': self.device_id,
            'device_type': self.device_type,
            'hardware_platform': self.hardware_platform,
            'management': {},
            'interfaces': {
                port_id: {
                    'physical_port': iface.physical_port,
                    'speed_class': iface.speed_class,
                    'media_type': iface.media_type,
                    'trex_port': iface.trex_port,
                }
                for port_id, iface in self.interfaces.items()
            },
        }
        if self.management_host:
            node['management']['host'] = self.management_host
        if self.management_port:
            node['management']['port'] = self.management_port
        if self.api_server:
            node['management']['api_server'] = self.api_server
        if self.username:
            node['management']['username'] = self.username
        if self.password:
            node['management']['password'] = self.password
        if self.sub_type:
            node['sub_type'] = self.sub_type
        return node

@dataclass
class UnifiedInterface:
    """归并后的单端口"""
    port_id: str                               # port1, port2, ...
    physical_port: Optional[str] = None        # 物理端口名（如 GE0_1）
    speed_class: Optional[str] = None          # GE | TE | XTE | QTE
    media_type: Optional[str] = None           # copper | fiber
    trex_port: Optional[str] = None            # TG TRex 端口号
    allocated_to: Optional[str] = None         # 占用标记（从台账取）
    allocated_vlan: Optional[int] = None       # 占用 VLAN

@dataclass
class PoolData:
    """物理池原始数据"""
    metadata: Dict[str, Any]
    nodes: Dict[str, Dict]                     # 原始 pool nodes
    links: List[Dict]

@dataclass
class DevicesData:
    """设备清单原始数据（S3-A：device_groups 实为 list，见 devices.yaml.example schema）

    遍历必须使用 list 迭代（for group in device_groups），不得用 .items()/.values()。
    组元素结构：{name, description, firewall:{host, device_type, ...},
                tg:{host, sub_type, hardware_platform, api_server, ...}, tags[]}
    """
    device_groups: List[Dict]                    # 原始 devices device_groups（list，S3-A）

@dataclass
class LedgerData:
    """台账原始数据（P0-3：保持现有 list + port_status schema，兼容 physical_pool.py）"""
    devices: List[Dict]                          # 台账 entries（list：[{device_id, brand, model, version, status, port_status}]）
    ledger_by_id: Dict[str, Dict] = field(default_factory=dict)  # device_id → entry 索引（load 时构建）
```

### 5.2 归并逻辑（关联键）

```
物理池节点                 devices.yaml（device_groups 为 list，S3-A）    台账（list + port_status，P0-3）
──────────────────────────────────────────────────────────────────
DUT: pool.nodes[FW1]      device_groups[].firewall     ledger.entries[device_id=201]
     .management.host ───► .host                           .model / .brand / .status
     .device_id (201) ───► ledger entry (device_id 关联)   .model ──► device-reference 平台等价类
     .device_type ←──────────────── (.device_type)         .port_status[].allocated_to / allocated_vlan
     .interfaces                                           （P2-5：model 与 device_type 可能同值直通）
                             补充: .version, .vendor
                             
TG:  pool.nodes[TG1]      device_groups[].tg           ledger.entries[device_id=207]
     .management.api_server──► .api_server                 .model / .status
     （api_server 归一化后匹配，P1-4/S3-C；host 兜底）       .port_status[].allocated_to / allocated_vlan
     .interfaces
                             补充: .sub_type (ixia-c/trex)
                                   .hardware_platform (EP/C236/J1900)
                                   
SW:  pool.nodes[SW1]      device_groups[].switch       （SW 通常无台账）
     仅校验 devices 存在性

PC:  pool.nodes[PC1]      device_groups[].client       （PC 通常无台账）
     仅校验 devices 存在性
```

### 5.3 台账三角映射

```
台账.model（完整 device_type，如 "DAS-TGFW-<MODEL>"，P2-5 同值直通）
    │
    ├─→ 池.device_type（如 "DAS-TGFW-A1600-HU"）
    │     关联键：device_id（pool node.device_id ↔ ledger entry.device_id，P0-3）
    │     直通规则：若台账 model 已是完整 device_type（DAS-TGFW-* 前缀）→ 直接采用；
    │               若为短型号（如 A1600-HU）→ 加规范前缀规范化
    │
    └─→ device-reference 平台等价类
          查表：device_type "DAS-TGFW-A1600-HU" → 硬件系列 "乐研海光3250"
                → hardware_platform 等价类 "HG3250"（P1-3，与 S2 _disambiguate_model 语义统一）

型号冲突消歧（S3-D）：A1500-HU 等歧义型号的消歧**统一在 S2 平台别名层**
（S2 `_disambiguate_model` + `platform_alias.yaml` conflicts，结合 devices 实际
hardware_platform 等价类）。S3 只消费 S2 已消歧的 device_type 集合，**不重复决策**；
歧义型号未被 S2 消歧时，S3 返回 None + warning（交 S2/调用方处理）。

映射结果回填到 UnifiedNode.hardware_platform（等价类取值）
```

**无持久化**：归并结果仅作为内存对象（最终池数据）传给 S4/S7 构造 raw `PhysicalPool`。不写回 physical_pool.yaml、devices.yaml 或 ledger 文件。

## 6. API / Interface 设计

### 6.1 `merge_pool(pool_path: str, devices_path: str, ledger_path: Optional[str] = None) -> UnifiedPool`

**主入口**。

**参数**：
- `pool_path`：目标项目 `topology/physical_pool.yaml` 路径
- `devices_path`：目标项目 `topology/devices.yaml` 路径
- `ledger_path`：目标项目 `topology/physical_ledger.yaml` 路径（可选，台账可能不存在）

**返回**：`UnifiedPool` 对象（最终池数据：全部归并后的节点 + links + `ledger_path`；S4/S7 用 `to_pool_data()` + `ledger_path` 构造 raw `PhysicalPool`，S3-B）

**异常**：
- `FileNotFoundError("physical_pool.yaml not found at {path}")` — 物理池文件不存在
- `FileNotFoundError("devices.yaml not found at {path}")` — 设备清单不存在
- `ValueError("YAML parse error in {path}")` — YAML 语法错误

### 6.2 `load_pool(pool_path: str) -> PoolData`

加载物理池 YAML，返回结构化的 `PoolData`。使用 `yaml.safe_load` + schema 前置校验（含 `nodes` + `links` 键）。

### 6.3 `load_devices(devices_path: str) -> DevicesData`

加载 devices.yaml。`device_groups` 为 **list**（S3-A，见 devices.yaml.example），每个组元素的块键支持：`firewall`（DUT）、`tg`（TG）、`switch`（SW）、`client`（PC）。

### 6.4 `load_ledger(ledger_path: str) -> LedgerData`

加载台账 YAML。`ledger_path` 为 `None` 或文件不存在时返回空 `LedgerData(devices={})` 并记录 warning。

### 6.5 内部关联函数

```python
def _find_device_group_by_host(host: str, devices_data: DevicesData) -> Optional[Dict]:
    """按 management.host 查找 DUT device_group（S3-A：device_groups 为 list，list 遍历）"""
    for group in devices_data.device_groups:
        fw = group.get('firewall', {})
        if fw.get('host') == host:
            return group
    return None

def _normalize_api_server(raw: Optional[str]) -> Optional[str]:
    """归一化 api_server（P1-4）：去 scheme、去尾斜杠、统一小写为 IP:port。

    例：pool 'http://10.113.55.161:8000' 与 devices '10.113.55.161:8000' 归一同为
        '10.113.55.161:8000'（devices.example / device-reference.md 契约：IP:端口）。
    """
    if not raw:
        return None
    s = str(raw).strip()
    if '://' in s:
        s = s.split('://', 1)[1]
    s = s.rstrip('/')
    return s.lower()

def _find_device_group_by_api_server(api_server: str, host: Optional[str],
                                     devices_data: DevicesData) -> Optional[Dict]:
    """按 management.api_server 查找 TG device_group（H2 + P1-4 归一化后精确匹配，S3-A list 遍历）。

    S3-C：**api_server 为关联主键**（归一化后精确匹配）；当池 TG 节点无 api_server
    或 devices tg 无 api_server 时，**回退 host 精确匹配（host 兜底）**。
    池 TG 节点既无 api_server 又无 host → 关联失败，记录 unmatched_nodes（非阻断）。
    """
    target = _normalize_api_server(api_server)
    for group in devices_data.device_groups:
        tg = group.get('tg', {})
        if target and _normalize_api_server(tg.get('api_server')) == target:
            return group
    if host:
        for group in devices_data.device_groups:
            tg = group.get('tg', {})
            if tg.get('host') and tg['host'] == host:
                return group
    return None
```

## 7. 核心处理流程

```
merge_pool(pool_path, devices_path, ledger_path)
 │
 ├─ 1. load_pool(pool_path) → PoolData
 │     ├─ yaml.safe_load
 │     ├─ 校验结构：必须含 nodes + links
 │     └─ 返回 PoolData(pool_id, nodes, links)
 │
 ├─ 2. load_devices(devices_path) → DevicesData
 │     ├─ yaml.safe_load
 │     └─ 返回 DevicesData(device_groups)
 │
 ├─ 3. load_ledger(ledger_path) → LedgerData
 │     ├─ ledger_path=None → LedgerData(devices={}) + warning
 │     └─ 返回 LedgerData(devices)
 │
 ├─ 4. 对每个 pool.nodes (node_key, node) 执行归并循环：
 │   │
 │   ├─ node.node_type == "DUT":
 │   │   ├─ host = node.management.host
 │   │   ├─ dg = _find_device_group_by_host(host, devices_data)
 │   │   ├─ 若 dg 存在 → _merge_dut(node_key, node, dg)
 │   │   │   ├─ 取 dg.firewall.device_type / version / vendor
 │   │   │   ├─ 冲突检测：pool.device_type vs dg.firewall.device_type
 │   │   │   │   ├─ 一致 → 合并（device_type 任取一方）
 │   │   │   │   └─ 不一致 → warning + 以 devices.device_type 为准
 │   │   │   ├─ 合并 interfaces：pool 端口 port<N> 映射到 UnifiedInterface
 │   │   │   └─ → UnifiedNode（node_id = node.get('node_id', node_key)，P1-5）
 │   │   └─ 若 dg 不存在 → 标记 unmatched，仍创建 UnifiedNode（仅 pool 数据）
 │   │
 │   ├─ node.node_type == "TG":
 │   │   ├─ api_server = node.management.api_server（经 _normalize_api_server 归一化后匹配，P1-4）
 │   │   ├─ host = node.management.get('host')                        # S3-C：host 兜底
 │   │   ├─ dg = _find_device_group_by_api_server(api_server, host, devices_data)
 │   │   ├─ 若 dg 存在 → _merge_tg(node_key, node, dg)
 │   │   │   ├─ 补充 dg.tg.sub_type（ixia-c/trex）
 │   │   │   ├─ 补充 dg.tg.hardware_platform（EP/C236/J1900）
 │   │   │   └─ → UnifiedNode（node_id = node.get('node_id', node_key)，P1-5）
 │   │   └─ 若 dg 不存在 → 标记 unmatched（池 TG 无 api_server 且无 host 时为唯一关联失败，S3-C）
 │   │
 │   ├─ node.node_type ∈ ["SW", "PC", "Mock"]:
 │   │   └─ _merge_sw_pc(node, devices_data)
 │   │       ├─ 尝试按 host 查找 device_group（switch/client）
 │   │       ├─ 找到 → 补充 device_type/hardware_platform
 │   │       └─ 未找到 → 标记 unmatched（非阻断）
 │   │
 │   └─ 未知 node_type → warning + 仅保留 pool 数据
 │
 ├─ 5. _resolve_ledger 对每个 UnifiedNode：
 │   │  对每个 node（携带对应 pool_node）：
 │   │    ├─ 按 pool_node.device_id 查 ledger.ledger_by_id（P0-3：list + device_id 关联）
 │   │    ├─ 提取 .model → node.ledger_model（P2-5：与 device_type 同值直通）
 │   │    ├─ 提取 .port_status[].allocated_to → node.interfaces[].allocated_to（port_status schema）
 │   │    ├─ 台账三角映射：model → device-reference 平台等价类（消费 S2 已消歧的 device_type 集合，S3-D）
 │   │    │   若 node.hardware_platform 为空：
 │   │    │     查 S2 platform_alias.yaml aliases 的 equiv_class（device_type → 等价类，P1-3）
 │   │    │     填充 node.hardware_platform
 │   │    │   歧义型号（conflicts）未被 S2 消歧 → 不重复决策，记录 warning（S3-D）
 │   │    └─ 记录 ledger 缺失项（未阻断）
 │   │
 │   └─ 返回 UnifiedPool
 │
 └─ 6. 返回 UnifiedPool(nodes, links, ledger_path, unmatched_nodes, merge_warnings)
         （最终池数据，S3-B；S4/S7 用 to_pool_data() + ledger_path 构造 raw PhysicalPool）
```

### 错误路径

| 步骤 | 失败条件 | 异常 |
|------|---------|------|
| 1 | pool_path 不存在 | FileNotFoundError |
| 1 | pool YAML 格式非法 | ValueError |
| 2 | devices_path 不存在 | FileNotFoundError |
| 2 | devices YAML 格式非法 | ValueError |
| 3 | ledger YAML 格式非法 | ValueError（非阻断，降级为空台账） |
| 4 | host 关联失败（DUT 无匹配） | 非阻断 → unmatched_nodes[] |
| 4 | api_server 关联失败（TG 无匹配） | 非阻断 → unmatched_nodes[] |
| 4 | 型号冲突（同 host 不同 device_type） | 非阻断 → merge_warnings[] |

## 8. 技术细节

### 8.1 DUT host 关联实现

```python
def _merge_dut(key: str, pool_node: Dict, device_group: Dict) -> UnifiedNode:
    fw = device_group.get('firewall', {})
    node = UnifiedNode(
        node_id=pool_node.get('node_id', key),   # 如 "FW1"（key 为 pool.nodes dict key，P1-5）
        node_type='DUT',
        device_id=pool_node.get('device_id'),    # 台账/池 device_id（P0-3）
        management_host=pool_node['management']['host'],
        management_port=pool_node['management'].get('port'),
        device_type=fw.get('device_type', pool_node.get('device_type')),
        hardware_platform=fw.get('hardware_platform'),
        sub_type=None,
        source='merged'
    )

    # 冲突检测
    pool_dt = pool_node.get('device_type')
    fw_dt = fw.get('device_type')
    warnings = []
    if pool_dt and fw_dt and pool_dt != fw_dt:
        warnings.append(
            f"model conflict for host {pool_node['management']['host']}: "
            f"pool={pool_dt}, devices={fw_dt}. Using devices value."
        )
        # 以 devices 为准（HLD §4.3 AG-3 推荐 A）
        node.device_type = fw_dt

    # 合并接口（只保留 pool 中的物理端口信息）
    for port_name, port_info in pool_node.get('interfaces', {}).items():
        node.interfaces[port_name] = UnifiedInterface(
            port_id=port_name,
            physical_port=port_info.get('physical_port'),
            speed_class=port_info.get('speed_class'),
            media_type=port_info.get('media_type'),
            trex_port=port_info.get('trex_port')
        )

    return node, warnings
```

### 8.2 TG api_server 关联实现（H2）

```python
def _merge_tg(key: str, pool_node: Dict, device_group: Dict) -> UnifiedNode:
    tg = device_group.get('tg', {})
    node = UnifiedNode(
        node_id=pool_node.get('node_id', key),   # key 为 pool.nodes dict key（P1-5）
        node_type='TG',
        device_id=pool_node.get('device_id'),    # 台账/池 device_id（P0-3）
        device_type=pool_node.get('device_type'),
        hardware_platform=tg.get('hardware_platform', pool_node.get('hardware_platform')),
        sub_type=tg.get('sub_type', pool_node.get('sub_type')),
        api_server=pool_node['management'].get('api_server'),
        management_host=pool_node['management'].get('host'),
        source='merged'
    )

    # 补充从 devices 获取的 sub_type 与 hardware_platform
    if not node.sub_type:
        node.sub_type = tg.get('sub_type')  # ixia-c or trex
    if not node.hardware_platform:
        node.hardware_platform = tg.get('hardware_platform')  # EP/C236/J1900

    # 合并接口
    for port_name, port_info in pool_node.get('interfaces', {}).items():
        node.interfaces[port_name] = UnifiedInterface(
            port_id=port_name,
            physical_port=port_info.get('physical_port'),
            speed_class=port_info.get('speed_class'),
            media_type=port_info.get('media_type'),
            trex_port=port_info.get('trex_port')
        )

    return node, []
```

### 8.3 台账三角映射实现

```python
def _resolve_ledger(node: UnifiedNode, pool_node: Dict, ledger_data: LedgerData,
                    platform_map: Dict) -> List[str]:
    """台账三角映射：ledger.model → pool.device_type → device-reference 平台等价类
    （P0-3：台账为 list + port_status schema，按 device_id 关联；P2-5：model 与 device_type 同值直通）"""
    warnings = []
    device_id = (pool_node or {}).get('device_id')
    ledger_entry = ledger_data.ledger_by_id.get(device_id) if device_id is not None else None
    if ledger_entry is None:
        warnings.append(f"no ledger entry for node '{node.node_id}' (device_id={device_id})")
        return warnings

    # 步骤 1：台账 model → device_type（P2-5：可能同值直通）
    node.ledger_model = ledger_entry.get('model')
    if not node.ledger_model:
        warnings.append(f"ledger entry for '{node.node_id}' missing 'model' field")

    # 步骤 2：device_type → hardware_platform 等价类（P1-3）
    if node.device_type and not node.hardware_platform:
        hw_platform = _resolve_hardware_platform(node.device_type, platform_map)
        if hw_platform:
            node.hardware_platform = hw_platform
        else:
            warnings.append(
                f"cannot resolve hardware_platform for device_type '{node.device_type}'"
            )

    # 步骤 3：台账端口占用（port_status schema）→ UnifiedInterface.allocated_to
    port_status = ledger_entry.get('port_status') or {}
    for port_name, port_entry in port_status.items():
        if port_name in node.interfaces:
            node.interfaces[port_name].allocated_to = port_entry.get('allocated_to')
            node.interfaces[port_name].allocated_vlan = port_entry.get('allocated_vlan')

    return warnings

def _resolve_hardware_platform(device_type: str, platform_map: Dict) -> Optional[str]:
    """查 S2 platform_alias.yaml：device_type → hardware_platform 等价类（P1-3/S3-D）。

    仅消费 S2 已消歧的 device_type 集合；对 conflicts 中歧义型号（如 A1500-HU）
    不做决策——依赖 S2 `_disambiguate_model` 消歧结果，或返回 None 并记录 warning
    （歧义需 S2 结合 devices hardware_platform 等价类消歧）。"""
    for platform_key, entry in platform_map.get('aliases', {}).items():
        if device_type in entry.get('device_types', []):
            # 返回 hardware_platform 等价类（如 HG3250），与 S2 _disambiguate_model 语义统一
            return entry.get('equiv_class', platform_key)
    return None
```

### 8.4 池全量承载降级（备选 B）

当物理池 nodes 已含完整信息（device_type + hardware_platform + management），devices.yaml 只用于校验：

```python
def _check_devices_existence(pool_node: Dict, devices_data: DevicesData) -> Optional[str]:
    """仅校验 devices 存在性，不补充信息"""
    host = pool_node.get('management', {}).get('host')
    api_server = pool_node.get('management', {}).get('api_server')

    for group in devices_data.device_groups:        # S3-A：device_groups 为 list，list 遍历
        # DUT: 按 host
        fw = group.get('firewall', {})
        if host and fw.get('host') == host:
            return None  # 存在，OK

        # TG: 按 api_server（归一化后匹配，P1-4/S3-C）；无 api_server 回退 host 兜底
        tg = group.get('tg', {})
        if api_server and _normalize_api_server(tg.get('api_server')) == _normalize_api_server(api_server):
            return None
        if host and tg.get('host') == host:
            return None

    return f"node '{pool_node.get('node_id', 'unknown')}' (host={host}) not found in devices.yaml"
```

### 8.5 关联失败不阻断

所有关联失败只在 `unmatched_nodes[]` 和 `merge_warnings[]` 中记录，不抛异常。调用方（S7 topo-planning CLI）可选择是否将 `unmatched_nodes` 非空作为 warning 输出或 `--strict` 模式下阻断。

## 9. 安全与性能设计

### 安全

- **凭据不暴露**：`password` 字段在 pool/devices 中已是占位符（P-1 改造后），归并时透传不解析
- **只读消费**：不修改 pool/devices/ledger 原始文件
- **YAML 安全解析**：使用 `yaml.safe_load`

### 性能

- 单次 `merge_pool` 调用：
  - 3 个 YAML 文件读取（~20KB 总量）
  - O(n) 归并遍历（n = 物理池节点数）
  - 预计 < 200ms

## 10. 测试设计

### 单元测试（`skills/topo-planning/tests/test_pool_merge.py`）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_merge_dut_host_association` | DUT host 关联：pool FW1 host=10.113.55.132↔devices firewall host | UnifiedNode 含 device_type/hardware_platform |
| `test_merge_tg_api_server_association` | TG api_server 关联：pool TG1 api_server=http://10.113.55.161:8000↔devices tg api_server=10.113.55.161:8000（归一化去 scheme 后匹配，P1-4） | UnifiedNode 含 sub_type/hardware_platform |
| `test_merge_tg_host_fallback` | 池 TG 节点无 api_server（仅 host）↔ devices tg.host 匹配（S3-C host 兜底） | 关联成功，UnifiedNode 含 sub_type/hardware_platform |
| `test_merge_tg_no_key_unmatched` | 池 TG 节点既无 api_server 又无 host | 标记 unmatched，不阻断（S3-C） |
| `test_tg_host_fallback_decision_table` | 显式决策表（Minor-1）：candidate[0] 的 api_server 不匹配但 host 匹配，candidate[1] 的 api_server 匹配 → 必须先完整遍历 api_server 主键命中 candidate[1]，**不得首组误命中** candidate[0] host 兜底 | 关联到 api_server 匹配的候选组（非首组 host 兜底）；无 api_server 且无 host 时 unmatched |
| `test_device_groups_list_traversal` | devices.device_groups 为 list（devices.yaml.example schema），归并遍历不抛错且能找到组（S3-A） | 关联正确，无「遍历不到任何组」缺陷 |
| `test_merge_sw_no_devices` | SW 节点未在 devices 注册 | unmatched_nodes 含 SW ID |
| `test_merge_model_conflict` | 同 host 不同 device_type（pool=DAS-TGFW-A1600-HU, devices=DAS-TGFW-A1300-HU） | warning + devices 值优先 |
| `test_merge_all_nodes` | 7 节点完整归并（FW1/FW2/SW1/SW2/PC1/PC2/TG1） | 7 UnifiedNode，2 unmatched（PC 若无 devices 注册） |
| `test_ledger_triangle_mapping` | 台账 model=DAS-TGFW-A1600-HU（同值直通）→ device_type → hardware_platform 等价类 | UnifiedNode.hardware_platform="HG3250"（P1-3） |
| `test_ledger_port_allocation` | 台账端口 allocated_to/vlan 回填 | UnifiedInterface.allocated_to/allocated_vlan 正确 |
| `test_ledger_missing` | 台账文件不存在 | LedgerData 空 + warning |
| `test_pool_full_承载_degradation` | 池节点信息完整，devices 仅校验 | unified_node.source="pool" |
| `test_missing_pool_file` | physical_pool.yaml 不存在 | FileNotFoundError |
| `test_missing_devices_file` | devices.yaml 不存在 | FileNotFoundError |
| `test_unified_pool_links_preserved` | 归并后 links 数据未丢失 | UnifiedPool.links 与 pool links 一致 |
| `test_multiple_tg_in_pool` | 池有多个 TG 节点 | 各自按 api_server 独立归并 |
| `test_empty_pool` | 池 nodes 为空 | UnifiedPool 空 nodes，无异常 |
| `test_unknown_node_type` | pool node_type="UNKNOWN" | warning + 保留原始数据 |

### Fixture 需求

- `tests/fixtures/pool_merge/pool_sample.yaml`：含 FW1/FW2/SW1/SW2/PC1/PC2/TG1 7 节点的简化物理池（端口精简，不含真实 IP）
- `tests/fixtures/pool_merge/devices_sample.yaml`：含 firewall/switch/tg/client device_groups（端口精简；**device_groups 为 list，与 devices.yaml.example 一致，S3-A**）
- `tests/fixtures/pool_merge/ledger_sample.yaml`：含 FW1/FW2 的台账条目 + 端口占用状态（**list + port_status schema，与 P-1 模板一致**，P0-3）
- `tests/fixtures/pool_merge/pool_conflict.yaml`：同 host 不同 device_type 的冲突池（用于冲突测试）
- `tests/fixtures/pool_merge/pool_full.yaml`：池信息完整的备选 B 测试数据

### 集成测试

- S10 将验证 `merge_pool` → `UnifiedPool` → `TopoMapper.match()` 的全链路

## 11. 实施步骤

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-S3-01 | 定义 `UnifiedPool`/`UnifiedNode`/`UnifiedInterface`/`PoolData`/`DevicesData`/`LedgerData` 数据类 + `UnifiedPool.to_pool_data()` / `UnifiedNode.to_pool_node()`（S3-B） | 1.5h | 数据模型 |
| TASK-S3-02 | 实现 `load_pool`/`load_devices`/`load_ledger`（3 文件加载 + schema 校验；`device_groups` 按 list 加载，S3-A） | 1.5h | 加载器 |
| TASK-S3-03 | 实现 `_find_device_group_by_host` / `_find_device_group_by_api_server`（list 遍历 + TG host 兜底，S3-A/S3-C） | 1h | 关联查找 |
| TASK-S3-04 | 实现 `_merge_dut`（host 关联 + 冲突处理 + 接口合并） | 1.5h | DUT 归并 |
| TASK-S3-05 | 实现 `_merge_tg`（api_server 归一化关联 + sub_type/hardware_platform 补充，P1-4） | 1h | TG 归并（H2） |
| TASK-S3-06 | 实现 `_merge_sw_pc`（仅校验 + 降级） | 0.5h | SW/PC 归并 |
| TASK-S3-07 | 实现 `_resolve_ledger`（台账三角映射 + port_status 占用回填，P0-3/P2-5） | 1.5h | 台账映射 |
| TASK-S3-08 | 实现 `merge_pool` 主入口（串联 1-7 + unmatched + warnings） | 1h | 核心入口 |
| TASK-S3-09 | 实现池全量承载降级路径 `_check_devices_existence` | 0.5h | 备选 B |
| TASK-S3-10 | 编写单元测试（18 用例 + 5 fixture 文件；含 device_groups list 遍历 + TG host 兜底） | 2h | test_pool_merge.py |
| TASK-S3-11 | 编写测试 fixture（5 文件） | 1h | 测试数据 |
| **合计** | | **~13h** | |

## 12. 风险、难点与预研建议

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| devices.yaml schema 变化导致 `_find_device_group_by_host` 查找失败 | 中 | 使用 `.get()` 嵌套访问 + fallback，不因新增字段崩溃；`device_groups` 按 **list** 遍历（S3-A），若结构重大变更，S10 测试先捕获 |
| host 字段格式不一致（IP vs hostname vs null） | 低 | host 关联使用精确字符串匹配（`==`）；若目标项目使用 hostname 而 pool 存 IP，关联失败不阻断，只记 unmatched |
| 台账 model 字段缺失导致三角映射不完整 | 中 | 台账路径只是辅助，缺失不阻断。UnifiedNode.ledger_model 留空并 warning |
| TG api_server 格式不一致（pool 带 scheme vs devices IP:port） | 中 | 关联前 `_normalize_api_server` 归一化（去 scheme / 统一 IP:port，P1-4）；**api_server 主键 + host 兜底（S3-C）**；池 TG 无 api_server 且无 host 时记 unmatched |
| TG 节点的 device_group 键名不统一（`tg` vs `traffic_generator`） | 低 | 优先查 `tg` 键，回退查 `traffic_generator` 键（兼容性处理） |
| 物理池中同一 host 对应多个 pool node（一台设备多个逻辑角色） | 低 | 当前 pool 设计每个 host 一个 node。若未来出现，需在归并时引入 `device_id` 精确匹配 |

**预研建议**：
1. 确认目标项目 `devices.yaml` 中 `device_groups` 的实际键名约定（firewall/tg/switch/client vs 其他命名）
2. 与 S4 topo_mapper 确认 `UnifiedNode` 输出格式是否可直接消费（避免 S3→S4 额外转换层）
3. 测试 `devices.yaml` 中 TG 节点 api_server 格式与 pool 侧 scheme 差异——已由 P1-4 归一化方案覆盖，实现时验证归一化后匹配率

## 13. 实现灰区与取舍记录

### 灰区 1：TG 关联键选择（api_server vs host）

**问题**：TG 节点在 pool 中有 `management.api_server`（仪表 API 地址），在 devices 中有 `tg.api_server`；但某些 TG 设备（如基于 PC 的 trex）可能只有 `host` 没有 `api_server`。且两处 api_server 格式不一致：pool 用 `http://10.113.55.161:8000`（带 scheme），devices.example / device-reference.md 契约用 `IP:端口`（如 `10.113.52.253:8450`，无 scheme）。

**选项**：
- A. 主键 `api_server`，回退 `host`；关联前对 api_server 归一化（去 scheme / 统一 IP:port）
- B. 仅用 `api_server`，不做归一化（scheme 格式差异会导致匹配失败）
- C. 仅按 host 关联（丢失 api_server 语义）

**决策（P1-4 + S3-C 关闭）**：选 A + 归一化。**api_server 为关联主键**：关联前调用 `_normalize_api_server`（去 scheme、去尾斜杠、小写统一为 IP:port），使 pool 与 devices 两侧格式对齐后再精确匹配；**host 为兜底**：池 TG 节点无 api_server（或 devices tg 无 api_server）时回退 host 精确匹配。池 TG 节点既无 api_server 又无 host → 关联失败，记 unmatched_nodes（非阻断）。

**显式决策表（S3-C 关闭）**：

| 池 TG 节点 | devices 侧候选 | 决策 |
|---|---|---|
| 有 `api_server` | 存在归一化后与目标相同的 `tg.api_server` | **api_server 主键命中**（第一遍遍历） |
| 有 `api_server` | 无 api_server 主键命中，但存在 `tg.host == 池 host` | **host 兜底命中**（第二遍遍历） |
| 有 `api_server` | api_server 与 host 两遍均无匹配 | unmatched_nodes（非阻断） |
| 无 `api_server`（仅 `host`） | 存在 `tg.host == 池 host` | **host 兜底命中** |
| 无 `api_server`（仅 `host`） | 无 host 匹配 | unmatched_nodes（非阻断） |
| 无 `api_server` 且无 `host` | — | unmatched_nodes（非阻断） |

> **关键约束（Minor-1 修订）**：决策必须**先完整遍历全部 candidates 找 api_server 主键**，**再完整遍历找 host 兜底**。不得在单个 candidate 循环内同时判 api_server 与 host（否则第一个「host 匹配但 api_server 不匹配」的候选会**首组误命中** host 兜底，遮蔽后续 api_server 主键候选）。实现语义与 §6.5 `_find_device_group_by_api_server` 完全一致。

```python
def _find_tg_device_group(pool_node, devices_data):
    api_server = pool_node['management'].get('api_server')
    host = pool_node['management'].get('host')
    target_api = _normalize_api_server(api_server)

    # 第一遍：api_server 主键（遍历全部 candidates，非首组即中，Minor-1）
    if target_api:
        for group in devices_data.device_groups:       # S3-A：device_groups 为 list
            tg = group.get('tg', {})
            if _normalize_api_server(tg.get('api_server')) == target_api:
                return group                          # api_server 主键命中（归一化后精确匹配）

    # 第二遍：host 兜底（遍历全部 candidates，非首组即中，S3-C）
    if host:
        for group in devices_data.device_groups:
            tg = group.get('tg', {})
            if tg.get('host') == host:
                return group                          # host 兜底命中

    # 两遍均未命中 → unmatched（由 merge_pool 标记）
    return None
```

**证据**：device-reference.md §TG 测试仪型号对照表 TG 节点同时有 `api_server` 和 `host` 字段；`api_server` 格式契约 = `IP:端口`（`10.113.52.253:8450`），与 pool 当前 `http://10.113.55.161:8000` 的 scheme 差异需归一化。

### 灰区 2：台账三角映射的 device-reference 数据来源

**问题**：`_resolve_hardware_platform` 需要查 device-reference.md 的完整映射表，但这是 Markdown 表格。

**选项**：
- A. 复用 S2 的 `platform_alias.yaml` 做 device_type → platform 查询
- B. 直接硬编码 `DEVICE_TYPE_TO_PLATFORM` 字典
- C. 实时解析 device-reference.md 的 Markdown 表

**决策**：选 A。复用 S2 已建的 `platform_alias.yaml`，通过 aliases 内 `device_types` + `equiv_class` 反查（P1-3 统一返回 hardware_platform 等价类）。可选新增 `reverse_lookup` 段作性能优化，但以 aliases 为单一数据源：

```yaml
# platform_alias.yaml 可选 reverse_lookup 段（以 aliases 的 device_types + equiv_class 生成，避免双份维护）
reverse_lookup:
  DAS-TGFW-A1300-HU: HG3250                  # hardware_platform 等价类（P1-3）
  DAS-TGFW-A1500-HU: [JL-HG, HG3250]          # 冲突型号 → 等价类列表（P1-3）
  DAS-TGFW-A1600-HU: HG3250
  ...
```

S3 优先遍历 `platform_map['aliases']` 反查（`entry['device_types']` 命中 → 返回 `entry['equiv_class']`）。**型号冲突消歧统一在 S2 平台别名层**（S2 `_disambiguate_model` + `platform_alias.yaml` conflicts，结合 devices 实际 hardware_platform 等价类），S3 只消费 S2 已消歧的 device_type 集合，**不重复决策**（S3-D）；歧义型号（如 A1500-HU）若未被 S2 消歧，S3 返回 None + warning，交 S2/调用方处理。

### 灰区 3：冲突处理优先级（pool vs devices 哪个为准）

**问题**：HLD §4.3 AG-3 推荐 A，但备选 B（池全量承载）场景下冲突处理策略不同。

**选项**：
- A. 始终以 devices 为准（推荐 A）
- B. 以 pool 为准（备选 B 场景）
- C. 通过参数 `conflict_resolution` 控制（"devices_first" | "pool_first"）

**决策**：选 C。`merge_pool` 增加可选参数 `conflict_resolution="devices_first"`（默认 A），`"pool_first"`（备选 B）。调用方（S7 CLI `--merge-strategy` 或默认）决定策略。

## 14. 回滚与发布策略

### 回滚

- `pool_merge.py` 为新增文件，回滚即删除
- 无其他模块被修改

### 发布

- S3 随 `topo-planning` skill 统一安装（S7）
- S3 依赖 P-1（physical_pool.yaml 模板 schema），P-1 必须先完成
- S4（topo_mapper 增强）消费 `UnifiedPool` 输出，S3 与 S4 接口需对齐

## 15. Definition of Done（DoD）

- [ ] `merge_pool` 对标准 pool + devices + ledger 输入正确归并
- [ ] DUT host 关联：pool `management.host` ↔ devices `firewall.host` 映射正确
- [ ] TG api_server 关联：pool `management.api_server` ↔ devices `tg.api_server` 映射正确（归一化 P1-4）；**api_server 主键 + host 兜底（S3-C）**，池 TG 无 api_server 且无 host 时记 unmatched
- [ ] **device_groups 按 list 遍历（S3-A）**：`for group in device_groups`，不得 `.items()`/`.values()`；遍历不到任何组视为缺陷，单测覆盖
- [ ] 台账三角映射：ledger.model → device_type → hardware_platform 等价类 查询链路完整（P0-3/P1-3/P2-5）；**歧义型号消歧统一在 S2，S3 不重复决策（S3-D）**
- [ ] 型号冲突：同 host 不同 device_type → warning + devices 优先
- [ ] 关联失败：unmatched_nodes[] 记录，不阻断
- [ ] 池全量承载降级：仅校验 devices 存在性
- [ ] TG 节点补充 sub_type + hardware_platform
- [ ] `UnifiedPool.to_pool_data()` 导出 final pool_data（physical_pool.yaml schema），`ledger_path` 透传——S4/S7 可经 `PhysicalPool.from_pool_data()` 构造 raw PhysicalPool（S3-B）
- [ ] 15+ 个单元测试全部通过
- [ ] YAML 安全解析使用 `yaml.safe_load`
