---
doc_type: LLD
story_id: "CR037-S1"
story_slug: "topology-collection"
cr_id: "CR-037"
tier: "A"
status: "draft"
created_at: "2026-08-05"
owner: "meta-dev"
source_hld: "process/changes/CR-037-HLD.md §4.1"
source_story: "process/stories/STORY-CR037-S1.md"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "skills/topo-planning/src/topology_collection.py 新建 + parser.py 依赖补齐"
---

# STORY-CR037-S1-LLD：topology_collection.py 详细设计

> **Tier**: A（新建核心解析模块）  
> **Shared Fragments**: `resource/network-topology/topology-collection.md`（共享只读）、`resource/network-topology/index.yaml`（共享只读）、`skills/topo-config/src/parser.py`（依赖补齐）  
> **Open Items**: 无  
> **修订记录**: v0.4（2026-08-05，meta-dev）—— CP5 综合审核 NEEDS_REWORK 修订：S1-A `list_topologies` 候选集改以 `_parse_collection_md` 解析出的 topo_id key 集合为准（非 index.yaml 顶层，index 仅登记 2 个 collection 级资源条目），返回 12 个 node* 拓扑（§3/§5/§6.3/§10/§11/§15）；S1-B §5 输入示例按 collection.md 真实字段分布修正——仅 `node2_dut1_tg1_link2` 含 speed_class/media_type/default_ip/mac 全字段，其余 11 个拓扑为简写 `{link: linkN}`，并注明缺省字段由推导/下游补充。
> **修订记录**: v0.5（2026-08-05，meta-dev）—— CP5 第二轮复审 R3 修订：§7 模糊搜索辅助旧表述残留修正——候选集明确为 `_parse_collection_md` 解析出的 12 个 node* topo_id（同步 §6.3，与 index.yaml 顶层无关），删除不存在的 `node4_dut1_tg1_link4` 示例，改用真实候选 `node2_dut1_tg1_link3/link2/link5`，并注明真实多节点/含 SW 拓扑名；§10 `test_fuzzy_suggestion` 用真实候选集（§7/§10/§15）。

---

## 0. 工程依据

本模块是 CR-037 topo-planning skill 的第一个数据层组件。工程依据源自 HLD §4.1（topology_collection 模块设计），CR-037 §事实核对结论 E3（collection.md 路径确认为 `resource/network-topology/topology-collection.md`），以及 parser.py 行 70 缺失依赖 `from topology_collection import find_topology_file`。顶层目标是为 topo_mapper 提供 12 个标准逻辑拓扑的 YAML 文件路径（P1-1：collection.md 实际 12 个 `## node*` 拓扑，最大 `node5_dut3_tg1_sw1_link6`），且 `find_topology_file` 返回 **YAML 文件路径**（P0-1，与 `parser.py` 的 filepath 消费契约一致：`detect_topology_format(filepath)` + `_parse_yaml_topology(filepath)`），覆盖全部测试组网场景。

---

## 1. 目标

新建 `skills/topo-planning/src/topology_collection.py`，解析 `resource/network-topology/topology-collection.md` 提取 12 个标准拓扑的 YAML 代码块，提供 `find_topology_file(topo_id)` 查找函数作为 topo-planning 和 topo-config 的共同入口，**返回该拓扑的 YAML 文件路径**（写入标准缓存路径，供 `parser.py` 的 `detect_topology_format(filepath)` + `_parse_yaml_topology(filepath)` 消费），补齐 `parser.py` 中 `from topology_collection import find_topology_file` 的缺失依赖。

核心价值：将逻辑拓扑描述（collection.md 的混合 Markdown+YAML 格式）转化为结构化 Python dict，使 topo_mapper 可消费为匹配输入。

## 2. Requirements

### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | `find_topology_file(topo_id)` 接受 `topo_id`（如 `node2_dut1_tg1_link3`），返回该拓扑的 **YAML 文件路径**（`str`，写入标准缓存路径；调用方按 `parser.py:106-116` 契约执行 `detect_topology_format` + `_parse_yaml_topology`） | P0 |
| FR2 | 解析 `topology-collection.md`，识别 `## {topo_id}` 标题后第一个 ` ```yaml ` 代码块 | P0 |
| FR3 | 消费 `resource/network-topology/index.yaml`（P0-2 决策）：校验 collection 级资源条目 `tgfw-topo-collection` 存在且其 `compatible_consumers` 含 `ptm-te`；`topo_id` 存在性以 `_parse_collection_md` 解析出的 key 集合为准，不要求 node* 顶层条目 | P0 |
| FR4 | 从每个节点的 `interfaces` 自动推导 `min_ports`（接口数量 = 最小端口需求，collection 无显式 `min_ports` 字段） | P0 |
| FR5 | 提供 `list_topologies()` 返回可用 topo_id 清单 | P1 |
| FR6 | 支持 `topo_id` 模糊匹配/建议（编辑距离 ≤2 时提示候选） | P2 |

### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 解析失败（缺文件/YAML 语法错误/topo_id 不存在）返回清晰错误消息，不使用裸 traceback |
| NFR2 | collection.md 大小约 3000 行，全量解析 < 100ms |
| NFR3 | 模块可被 `topo-planning/src/` 和 `topo-config/src/` 同时 import（共享路径优先级处理） |
| NFR4 | 不修改 `resource/network-topology/` 下任何文件（共享只读资源） |

## 3. 模块拆分与职责

模块本身不拆子模块，但提供四个正交函数：

| 函数 | 职责 | 输入 | 输出 |
|------|------|------|------|
| `find_topology_file(topo_id)` | 主入口：查 topology_id → 返回 YAML 文件路径 | `topo_id: str` | `str`（YAML 文件路径，写入缓存） |
| `_parse_collection_md(md_path)` | 解析 collection.md 全文为 `{topo_id: yaml_dict}` | `md_path: str` | `Dict[str, Dict]` |
| `_load_index(index_path)` | 加载并校验 index.yaml | `index_path: str` | `Dict` |
| `derive_min_ports(topo_dict)` | 从 nodes.interfaces 推导每节点 min_ports | `topo_dict: Dict` | `Dict[node_id, int]` |
| `list_topologies(collection_path)` | 列出全部可用 topo_id（候选集 = `_parse_collection_md` 解析出的 topo_id key 集合，S1-A） | `collection_path: Optional[str]` | `List[str]`（12 个 node* topo_id） |

## 4. 代码结构与文件影响范围

### 新增文件

```
skills/topo-planning/src/topology_collection.py   # 主模块（~250 行）
```

### 修改文件（补齐依赖）

```
skills/topo-config/src/parser.py                   # 行 70：from topology_collection import find_topology_file
                                                   # 可正常 import（安装后 topo-planning/src 在 PYTHONPATH）
```

### 不修改 / 只读消费

```
resource/network-topology/topology-collection.md   # 只读消费
resource/network-topology/index.yaml               # 只读消费
```

### 路径定位策略

`topology_collection.py` 需在以下路径查找 collection.md：

1. `{SKILL_ROOT}/resource/network-topology/topology-collection.md`（相对于 topo-planning skill 根目录）
2. `os.path.expanduser("~/.ptm-team/resource/network-topology/topology-collection.md")`（共享资源安装路径，层2）
3. 按 1→2 优先级查找，均不存在则 `FileNotFoundError` + 提示运行 `ptm-team install resource`

## 5. 数据模型与持久化设计

本模块**无持久化**。输入/输出均为 Python 内存对象。

### 输入格式（collection.md YAML 块）

collection.md 中 12 个 `## node*` 拓扑的 YAML 块存在**两种数据形态**（S1-B，以仓库 `resource/network-topology/topology-collection.md` 实况为准）：

| 数据形态 | 拓扑 | 接口字段 | link 字段 |
|---------|------|---------|-----------|
| 全量明细 | `node2_dut1_tg1_link2`（唯一 1 个） | `speed_class` / `media_type` / `default_ip` / `default_ipv6` / `mac_address` / `chassis_id` / `board_id` / `port_id` / `link` | `media_type` / `mode` / `endpoints` / `network`(ipv4/ipv6/vlan) |
| 简写 | 其余 11 个（含 `node2_dut1_tg1_link3`） | 仅 `{link: linkN}` | 仅 `endpoints` |

**简写形态的真实示例（`node2_dut1_tg1_link3`，与 collection.md §对应 YAML 完全一致）**：

```yaml
metadata:
  version: "0.2"
  topology_id: node2_dut1_tg1_link3
  description: 三链路直连，用于吞吐扩展测试。

nodes:
  tg1:
    logic_id: tg1
    node_type: TG
    sub_type: IXIA
    interfaces:
      port1: {link: link1}
      port2: {link: link2}
      port3: {link: link3}
  dut1:
    logic_id: dut1
    node_type: DUT
    interfaces:
      port1: {link: link1}
      port2: {link: link2}
      port3: {link: link3}

links:
  link1:
    endpoints:
      - {node: tg1, interface: port1}
      - {node: dut1, interface: port1}
  link2:
    endpoints:
      - {node: tg1, interface: port2}
      - {node: dut1, interface: port2}
  link3:
    endpoints:
      - {node: tg1, interface: port3}
      - {node: dut1, interface: port3}
```

**字段缺失与补充说明（S1-B）**：

- collection.md **无 `min_ports` 字段** → 由 `derive_min_ports` 从每节点 `interfaces` 接口数推导（§6.2，接口数 = 端口需求）。
- 除 `node2_dut1_tg1_link2` 外，**接口无 `speed_class`/`media_type`、link 无 `network`** → 逻辑 topo 层缺省这些物理属性；映射阶段由 S2 用户 topo-limit 或 S4 物理池实际端口补齐，**S1 不虚造字段**（只消费 collection.md 真实存在的数据）。
- 个别拓扑 `metadata.notes` 说明「源图实际 N 条 link，topology_id 中 linkN 保留原始命名」（如 `node3_dut2_tg1_link6` 实际 5 条、`node4_dut2_tg1_sw2_link12` 实际 8 条）——解析器以 YAML `links` 实际条数为准，topo_id 中的 linkN 只是命名，不参与 link 数推导。

### 输出格式（`find_topology_file` 返回）

返回 **YAML 文件路径**（`str`）。本模块从 collection.md 提取该 topo_id 对应的 YAML 代码块后，`yaml.safe_dump` 写入标准缓存路径（`~/.ptm-team/cache/topology/{topo_id}.yaml`，见 §6.1），并返回该路径。

**消费契约（P0-1，与 `parser.py:70-116` 对齐）**：

```python
filepath = find_topology_file(topo_id)          # 返回路径（str）
fmt = detect_topology_format(filepath)           # '.yaml' → 'yaml'
return _parse_yaml_topology(filepath)            # → Topology 对象
```

本模块不依赖 `topology_model.py`，保持轻量。调用方（parser.py 或 topo-planning CLI）对返回路径做格式检测与 `_parse_yaml_topology` 转换。

### 辅助数据结构

```python
# _load_index 返回（P0-2 决策：只校验 collection 级资源条目，不要求 node* 顶层条目）
{
    "tgfw-topo-spec": {
        "resource_type": "network-topology",
        "compatible_consumers": ["ptm-tde", "ptm-te"],
        "status": "active",
        ...
    },
    "tgfw-topo-collection": {
        "resource_type": "network-topology",
        "compatible_consumers": ["ptm-tde", "ptm-te"],
        "status": "active",
        ...
    }
}
# 校验规则（P0-2）：
#   1. index.yaml 中必须存在 'tgfw-topo-collection' 条目，且其 compatible_consumers 含 'ptm-te'
#   2. 请求的 topo_id 必须存在于 _parse_collection_md 解析出的 key 集合中
#   3. 不要求 node* topo_id 出现在 index.yaml 顶层（index 只登记 collection 级资源）

# derive_min_ports 返回
{"tg1": 3, "dut1": 3}
```

## 6. API / Interface 设计

### 6.1 `find_topology_file(topo_id: str, collection_path: Optional[str] = None, index_path: Optional[str] = None) -> str`

**主入口**。按 `topo_id` 从 topology-collection.md 提取对应 YAML 拓扑，写入标准缓存路径并返回该路径。

**参数**：
- `topo_id`：标准 topology_id（如 `node2_dut1_tg1_link3`）
- `collection_path`：collection.md 显式路径（可选，默认按优先级查找）
- `index_path`：index.yaml 显式路径（可选）

**返回**：`str` — YAML 文件路径（写入 `~/.ptm-team/cache/topology/{topo_id}.yaml`；文件名带 topo_id，保证幂等覆盖）

**缓存写入流程**：
1. `_parse_collection_md` 解析出的 `{topo_id: yaml_dict}` 命中后，将 `yaml.safe_dump(yaml_dict)` 写入缓存路径
2. 目录不存在时自动创建（`os.makedirs(..., exist_ok=True)`）
3. 返回该文件路径，供调用方 `detect_topology_format` + `_parse_yaml_topology` 消费（P0-1）

**异常**：
- `FileNotFoundError("未找到 topology-collection.md...")` — collection.md 不存在
- `FileNotFoundError("未找到 topology_id='{topo_id}' 的拓扑")` — topo_id 不在 collection.md 解析出的 key 集合中
- `ValueError("collection 资源 tgfw-topo-collection 不在 index.yaml 中")` — index.yaml 无 collection 条目（P0-2）
- `ValueError("topology 资源 'tgfw-topo-collection' 的 compatible_consumers 不含 ptm-te")` — collection 条目 compatible_consumers 不含 ptm-te（P0-2）
- `ValueError("YAML parse error in collection.md at line {n}")` — YAML 语法错误

### 6.2 `derive_min_ports(topo_dict: Dict[str, Any]) -> Dict[str, int]`

从 topo_dict 的 `nodes` 中提取每节点的接口数作为 `min_ports`。

**语义**：若节点已显式声明 `min_ports`，使用声明值；否则 `len(interfaces)`。

**返回**：`{node_id: min_ports}`

### 6.3 `list_topologies(collection_path: Optional[str] = None) -> List[str]`

返回全部可用 topo_id 清单（候选集 = `_parse_collection_md` 解析出的 topo_id key 集合，S1-A）。

- **候选集来源是 collection.md 的 `## node*` 标题集合，不是 index.yaml 顶层**：index.yaml 只登记 2 个 collection 级资源条目（`tgfw-topo-spec` / `tgfw-topo-collection`），无 node* 顶层条目（P0-2）。
- 返回 12 个 node* topo_id（`node2_dut1_tg1_link2` ~ `node3_dut1_mock1_link2`，P1-1），顺序按 collection.md 出现顺序。
- 参数 `collection_path` 显式传入时跳过默认路径查找；缺省按 §6.1 优先级查找（`_locate_collection_md`）。
- 本函数**不返回 index.yaml 的资源条目**（display_name/description 等摘要由 CLI 层按需从 index 加载展示，不作为候选集）。

### 6.4 parser.py 补齐依赖

```python
# parser.py 行 70（当前代码）
from topology_collection import find_topology_file

# 安装后路径可及条件：
# 统一 PYTHONPATH 策略（P2-4）：install.py（S8）将 skills/topo-planning/src/ 加入 PYTHONPATH，
# 与 topo-config/src/ 一并管理（对齐 parser.py:15-17 的 sys.path 策略）。S1 不引入 importlib 临时代码。
```

**消费契约（P0-1）**：`parser.parse_topology(topo_id=...)` 分支：

```python
filepath = find_topology_file(topo_id)   # 返回 YAML 文件路径（str）
if not filepath:
    raise FileNotFoundError(...)
fmt = detect_topology_format(filepath)    # '.yaml' → 'yaml'
return _parse_yaml_topology(filepath)     # 与 parser.py:106-116 契约一致
```

**过渡降级**（仅当 S8 统一 PYTHONPATH 尚未交付时）：`parse_topology` 中通过 `sys.path.insert(0, <topo-planning/src>)` 补路径，不使用 `importlib.util` 临时代码（P2-4）。

## 7. 核心处理流程

```
find_topology_file(topo_id)
 │
 ├─ 1. _locate_collection_md()  ─── 按优先级查 collection.md 路径
 │   ├─ 层1: {SKILL_ROOT}/resource/network-topology/topology-collection.md
 │   └─ 层2: ~/.ptm-team/resource/network-topology/topology-collection.md
 │
 ├─ 2. _load_index(index_path)  ─── 加载 index.yaml（P0-2 决策）
 │   ├─ 校验 'tgfw-topo-collection' 条目存在且 compatible_consumers 含 ptm-te
 │   └─ 不要求 node* topo_id 出现在 index.yaml 顶层
 │
 ├─ 3. _parse_collection_md(collection_path)  ─── 全文解析
 │   │  逐行扫描 collection.md
 │   │  状态机: SCANNING → IN_HEADER(匹配 ## topo_id) → IN_YAML(收集代码块行)
 │   │  遇到下一个 ## 或 EOF → 结束当前 YAML 块收集
 │   │  收集到 {topo_id: yaml_str} 字典后 yaml.safe_load 解析
 │   ├─ 命中 12 个拓扑 YAML（node2_dut1_tg1_link2 ~ node3_dut1_mock1_link2，P1-1）
 │   └─ topo_id 不在 key 集合 → FileNotFoundError + 建议 find_similar_topo_ids(topo_id)
 │
 ├─ 4. derive_min_ports(topo_dict)  ─── 推导每节点 min_ports
 │   对 nodes 中每个 node:
 │   if node.get('min_ports'): return node['min_ports']
 │   else: return len(node.get('interfaces', {}))
 │
 └─ 5. 将 topo_dict 写入缓存路径 ~/.ptm-team/cache/topology/{topo_id}.yaml
     （含推导后的 min_ports，注入 topo_dict['nodes'][node_id]['min_ports']）
     并返回该文件路径（str，P0-1）
```

### 错误路径

| 步骤 | 失败条件 | 异常类型 | 错误消息 |
|------|---------|---------|---------|
| 1 | collection.md 不存在 | `FileNotFoundError` | "未找到 topology-collection.md。请运行 `ptm-team install resource` 安装共享资源。" |
| 2 | index.yaml 无 tgfw-topo-collection 条目 | `ValueError` | "collection 资源 tgfw-topo-collection 不在 index.yaml 中" |
| 2 | tgfw-topo-collection 的 compatible_consumers 不含 ptm-te | `ValueError` | "topology 资源 'tgfw-topo-collection' 的 compatible_consumers 不含 ptm-te" |
| 3 | YAML 解析失败 | `ValueError` | "YAML parse error in collection.md at line {n}: {detail}" |
| 3 | topo_id 不在 collection.md 解析出的 key 集合 | `FileNotFoundError` | "未找到 topology_id='{id}'。可用拓扑：\n - {topo_1}\n - {topo_2}..." |

### 模糊搜索辅助

`_find_similar_topo_ids(topo_id, candidates, max_distance=2) → List[str]`：当精确匹配失败时，对**候选集（`_parse_collection_md` 解析出的 12 个 node* topo_id，S1-A）**计算编辑距离，返回距离 ≤2 的候选。候选集**与 index.yaml 顶层无关**（index 只登记 2 个 collection 级资源条目，不逐条登记 node* 拓扑，P0-2）。如 `node2_dut1_tg1_link3` 误输入为 `node2_dut1_tg1_link4` 时，编辑距离 ≤2 的真实候选为 `node2_dut1_tg1_link3, node2_dut1_tg1_link2, node2_dut1_tg1_link5`（三者均存在于 collection.md；**不存在 `node4_dut1_tg1_link4`** 拓扑，真实多节点/含 SW 拓扑为 `node4_dut3_tg1_link7` / `node4_dut2_tg1_sw2_link7` / `node4_dut2_tg1_sw1_link6` / `node4_dut2_tg1_sw2_link12`）。

## 8. 技术细节

### 8.1 Markdown YAML 提取状态机

```python
def _parse_collection_md(md_path: str) -> Dict[str, Dict]:
    """
    逐行扫描 collection.md：
    - 遇到 `## {word}` → 检查是否匹配 topo_id 模式（如 `node2_dut1_tg1_link2`）
    - 匹配后进入 IN_YAML 状态，收集 ` ```yaml ` 后到 ` ``` ` 前的所有行
    - 遇到下一个 `## ` 或 EOF → 结束当前块
    """
    topo_pattern = re.compile(r'^##\s+(node\d+_.+)$')  # 匹配 topo_id 模式
    results = {}
    current_topo_id = None
    in_yaml = False
    yaml_lines = []

    with open(md_path, 'r', encoding='utf-8') as f:
        for line_no, line in enumerate(f, 1):
            # 检测 ## header
            header_match = topo_pattern.match(line)
            if header_match:
                if current_topo_id and yaml_lines:
                    results[current_topo_id] = yaml.safe_load('\n'.join(yaml_lines))
                current_topo_id = header_match.group(1)
                yaml_lines = []
                in_yaml = False
                continue

            # 检测 YAML 块开始
            if current_topo_id and not in_yaml and line.strip() == '```yaml':
                in_yaml = True
                continue

            # 检测 YAML 块结束
            if in_yaml and line.strip() == '```':
                in_yaml = False
                continue

            # 收集 YAML 行
            if in_yaml:
                yaml_lines.append(line)

    # 处理文件末尾最后一个块
    if current_topo_id and yaml_lines:
        results[current_topo_id] = yaml.safe_load('\n'.join(yaml_lines))

    return results
```

**关键决策**：
- **不收集 Mermaid 块**：状态机只有进入 ` ```yaml ` 后才收集，` ```mermaid ` 被忽略。
- **topo_id 模式匹配**：使用正则 `^##\s+(node\d+_.+)$`，覆盖 12 个拓扑命名约定（如 `node2_dut1_tg1_link2` 到 `node5_dut3_tg1_sw1_link6`；最大为 node5，不存在 node8 拓扑，P1-1）。
- **不依赖特定 YAML 库**：仅使用 `yaml.safe_load`，与 parser.py 一致。

### 8.2 index.yaml 消费

```python
def _load_index(index_path: str) -> Dict:
    """加载 index.yaml 为 {topo_id: entry}，并校验 collection 级资源条目（P0-2 决策）。"""
    with open(index_path, 'r', encoding='utf-8') as f:
        index = yaml.safe_load(f)

    # 转为 {topo_id: entry} 索引（index 只登记 collection 级资源：tgfw-topo-spec / tgfw-topo-collection）
    topo_index = {}
    for entry in index.get('topologies', []):
        tid = entry.get('topo_id')
        if tid:
            topo_index[tid] = entry

    # P0-2 校验：collection 资源条目必须存在且 compatible_consumers 含 ptm-te
    collection_entry = topo_index.get('tgfw-topo-collection')
    if not collection_entry:
        raise ValueError("collection 资源 tgfw-topo-collection 不在 index.yaml 中")
    _check_compatible(collection_entry, 'ptm-te')
    return topo_index

def _check_compatible(topo_entry: Dict, consumer: str = 'ptm-te') -> None:
    if consumer not in topo_entry.get('compatible_consumers', []):
        raise ValueError(
            f"topology 资源 '{topo_entry.get('topo_id')}' 的 compatible_consumers 不含 {consumer}"
        )
```

### 8.3 parser.py 依赖补齐

当前 parser.py 行 70：
```python
from topology_collection import find_topology_file
```

这段代码已在 `parse_topology(topo_id=...)` 分支中执行。补齐措施（P2-4 决策：统一 PYTHONPATH，不引入 importlib 临时代码）：

1. **路径策略**：topo-planning skill 安装到 `~/.ptm-team/skills/topo-planning/` 后，安装器（S8）将 `skills/topo-planning/src/` 加入 `PYTHONPATH`，与 `topo-config/src/` 一并管理（对齐现有 `parser.py:15-17` 的 sys.path 策略）。

2. **消费契约（P0-1）**：
```python
filepath = find_topology_file(topo_id)   # 返回 YAML 文件路径（str）
fmt = detect_topology_format(filepath)    # '.yaml' → 'yaml'
return _parse_yaml_topology(filepath)
```

3. **过渡降级**（仅当 S8 尚未交付统一 PYTHONPATH）：`parse_topology` 内 `sys.path.insert(0, os.path.join(ROOT_DIR, '../../topo-planning/src'))`，不使用 `importlib.util` 临时代码。

最终统一 import 策略由 S7/S8 收束为「安装器统一管理 PYTHONPATH」。S1 只保证模块独立可用 + 返回路径契约正确，import 路径由 S8 统一注入。

### 8.4 min_ports 推导

```python
def derive_min_ports(topo_dict: Dict[str, Any]) -> Dict[str, int]:
    nodes = topo_dict.get('nodes', {})
    result = {}
    for node_id, node in nodes.items():
        if 'min_ports' in node:
            result[node_id] = node['min_ports']
        else:
            interfaces = node.get('interfaces', {})
            result[node_id] = len(interfaces)
    return result
```

注入到返回的 topo_dict 中：`topo_dict['nodes'][node_id]['min_ports'] = derived_value`（不覆盖已有 `min_ports` 显式声明值）。

## 9. 安全与性能设计

### 安全

- **只读消费**：不修改 `resource/network-topology/` 下任何文件
- **无凭据暴露**：collection.md 中的 `password: "<password>"` 为占位符，本模块不透传敏感数据
- **YAML 安全解析**：使用 `yaml.safe_load` 而非 `yaml.load`，防止代码注入

### 性能

- **惰性解析**：首次调用 `find_topology_file` 时解析 collection.md 全文并缓存，后续调用直接查缓存
- **缓存策略**：模块级 `_collection_cache: Dict[str, Dict] = {}`，用 `md_path` + `mtime` 做缓存键
- **预估性能**：3000 行 collection.md 全文 scan + 12 个 YAML 块解析 < 50ms

## 10. 测试设计

### 单元测试（`skills/topo-planning/tests/test_topology_collection.py`）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_find_existing_topo` | `find_topology_file("node2_dut1_tg1_link3")` 返回 YAML 文件路径 | 路径存在、`.yaml` 后缀、`detect_topology_format` → `yaml` |
| `test_find_nonexistent_topo` | `find_topology_file("no_such_topo")` | `FileNotFoundError` + 候选建议 |
| `test_missing_collection_entry` | 模拟 index.yaml 无 tgfw-topo-collection 条目 | `ValueError` |
| `test_compatible_consumers_check` | 模拟 tgfw-topo-collection 的 compatible_consumers 不含 ptm-te | `ValueError` |
| `test_missing_collection_md` | collection.md 路径不存在 | `FileNotFoundError` + install 提示 |
| `test_yaml_parse_error` | 注入非法 YAML 的 fixture collection.md | `ValueError` + 行号 |
| `test_derive_min_ports` | `derive_min_ports({'nodes': {'tg1': {'interfaces': {'port1': {}, 'port2': {}}}}})` | `{'tg1': 2}` |
| `test_derive_min_ports_explicit` | 节点显式有 `min_ports: 4` | 4（不推导） |
| `test_all_12_topos` | 遍历 `_parse_collection_md` 解析出的全部 12 个 topo_id（P1-1） | 命中率 100%，全部成功返回 YAML 文件路径（P0-2） |
| `test_list_topologies_returns_12` | `list_topologies()` 候选集来自 collection.md 而非 index.yaml（S1-A） | 返回集合 == `_parse_collection_md` 的 key 集合；长度 = 12；含 `node2_dut1_tg1_link2` 与 `node3_dut1_mock1_link2`；不含 index 顶层 `tgfw-topo-spec`/`tgfw-topo-collection` |
| `test_data_shape_distribution` | 抽查 12 个 YAML 块字段分布（S1-B）：仅 `node2_dut1_tg1_link2` 接口含 speed_class/media_type，其余 11 个为简写 `{link: linkN}` | 与 collection.md 实况一致；简写拓扑接口不含 speed_class/media_type，不虚造字段 |
| `test_fuzzy_suggestion` | `find_topology_file("node2_dut1_tg1_link4")` 触发模糊搜索 | 候选集来自 `_parse_collection_md`（12 个 node*，非 index.yaml）；编辑距离 ≤2 的真实候选 = `{node2_dut1_tg1_link3, node2_dut1_tg1_link2, node2_dut1_tg1_link5}`（均存在，不含不存在的 `node4_dut1_tg1_link4`，R3） |
| `test_cache_hit` | 同 topo_id 多次调用 | 第二次调用不重新读文件（通过 mock 验证） |

### 集成测试

- S10 将验证 `parser.parse_topology(topo_id="node2_dut1_tg1_link3")` 完整链路（collection → Topology 对象）

### Fixture 需求

- S1 测试需要 resource 目录中有真实的 `topology-collection.md` 与 `index.yaml`（仓库已有）
- 需要 1 个 `fixture_collection_broken.md`（故意引入 YAML 语法错误）用于错误路径测试
- 需要 1 个 `fixture_index_no_ptm_te.yaml`（compatible_consumers 不含 ptm-te）用于校验路径测试

## 11. 实施步骤

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-S1-01 | 创建 `topology_collection.py` 骨架（函数签名 + 路径查找 `_locate_collection_md`） | 30min | 模块骨架 |
| TASK-S1-02 | 实现 `_parse_collection_md` — Markdown YAML 提取状态机 | 1h | 核心解析逻辑 |
| TASK-S1-03 | 实现 `_load_index` — index.yaml 加载与校验 | 30min | 索引消费 |
| TASK-S1-04 | 实现 `find_topology_file` — 主入口串联 1-3 步 + 缓存写入返回路径 | 30min | 核心 API |
| TASK-S1-05 | 实现 `derive_min_ports` — 端口需求推导 | 30min | 辅助函数 |
| TASK-S1-06 | 实现 `list_topologies`（候选集 = `_parse_collection_md` key 集合，S1-A）+ 模糊搜索 `_find_similar_topo_ids` | 30min | 辅助 API |
| TASK-S1-07 | 补齐 parser.py 中 import 路径（`sys.path.insert`，统一 PYTHONPATH 策略） | 1h | 依赖补全 |
| TASK-S1-08 | 错误处理 + 错误消息（5 种错误路径） | 1h | 完善错误处理 |
| TASK-S1-09 | 单元测试编写（12 用例 + 3 fixture，含 `test_list_topologies_returns_12` / `test_data_shape_distribution`） | 1.5h | test_topology_collection.py |
| TASK-S1-10 | 缓存机制实现 + 测试 | 30min | 性能优化 |
| **合计** | | **~7h** | |

## 12. 风险、难点与预研建议

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| collection.md 格式与状态机预期不符（如 ## 标题非 topo_id 格式） | 低 | 状态机正则 `^##\s+(node\d+_.+)$` 白名单匹配，非 topo_id 标题自动跳过 |
| 12 个拓扑中两种数据形态并存（1 个全量明细 + 11 个简写） | 中 | 已核对 collection.md 实况（S1-B）：状态机只提取 YAML 块本身，不要求统一字段；字段缺失由 §5 缺省说明 + S2/S4 下游补充，S1 不虚造 |
| parser.py import 路径在不同安装环境下不可及 | 中 | S7/S8 决定统一 PYTHONPATH 策略（P2-4），S1 提供 sys.path 过渡降级，不引入 importlib 临时代码 |
| index.yaml 结构扩展（新增字段）导致 _load_index 解析不完整 | 低 | 使用 `.get()` 访问 + 向后兼容字段，不因未知字段崩溃 |
| 缓存 mtime 检测在多进程/并发场景下失效 | 低 | collection.md 是只读共享资源，运行时不变，可简单缓存。若需刷新，提供 `clear_cache()` |

**预研建议**：
1. 运行 `python topology_collection.py` 对全部 12 个拓扑做 dry-run 解析，确保格式兼容
2. 收集到的 12 个 YAML 块分别运行 `yaml.safe_load` + `parser._parse_yaml_topology` 验证 schema 兼容性

## 13. 实现灰区与取舍记录

### 灰区 1：topo_id 模式匹配范围

**问题**：collection.md 中 `## ` 标题不仅有 topo_id（如 `## node2_dut1_tg1_link2`），还有通用标题（如 `## 通用 Mermaid 样式`）。

**选项**：
- A. 正则白名单 `^##\s+(node\d+_.+)$` 仅匹配已知命名模式
- B. 收集所有 `## ` 后 ` ```yaml ` 块，从 YAML 内 `metadata.topology_id` 反查

**决策**：选 A。collection.md 命名约定稳定（`node<节点数>_dut<N>_tg<N>_...link<N>`），白名单匹配防污染。若未来扩展新命名模式，更新正则或提供 `topo_pattern` 参数。

**证据**：`topology-collection.md` 中 `## node*` 标题共 12 个（node2 ~ node5，最大 `node5_dut3_tg1_sw1_link6`，P1-1）；`index.yaml` 只登记 2 个 collection 级条目（`tgfw-topo-spec` / `tgfw-topo-collection`），不逐条登记 node* 拓扑（P0-2）。

### 灰区 2：parser.py import 策略

**问题**：`topo-config/src/parser.py` 行 70 直接 `from topology_collection import find_topology_file`，但两个 skill 在不同目录树。

**选项**：
- A. 安装器统一管理 PYTHONPATH（`~/.ptm-team/skills/*/src/` 加入 path）
- B. parser.py 内 `importlib.util` 动态加载，不依赖安装器（临时代码，S8 后需返工）
- C. 将 `find_topology_file` 复制到 topo-config/src/ 内作为共享模块（重复维护）

**决策（P2-4 修订）**：选 A，与 install.py 统一 PYTHONPATH 策略对齐。S1 只保证模块独立可用 + 返回路径契约正确；S8 安装器将 `topo-planning/src/` 加入 `PYTHONPATH`。过渡期 parser.py 使用 `sys.path.insert(0, <topo-planning/src>)` 补路径，**不引入 importlib 临时代码**，避免 S7/S8 返工删除。

**重访条件**：S8 确定统一 import 策略后，移除 parser.py 中 sys.path 过渡代码。

## 14. 回滚与发布策略

### 回滚

- `topology_collection.py` 是新增文件，回滚即删除该文件
- parser.py 的 `from topology_collection import find_topology_file` 在 S1 前已存在（当前会 ImportError），回滚后状态不变
- `tests/test_topology_collection.py` + fixtures 一并删除

### 发布

- S1 作为 `topo-planning` skill 的一部分随 S7 统一安装
- S1 模块独立于 S2-S6（无内部依赖），可提前交付测试

## 15. Definition of Done（DoD）

- [ ] `find_topology_file(topo_id)` 对全部 12 个拓扑返回合法 YAML 文件路径（P0-1/P1-1）
- [ ] `derive_min_ports` 正确推导每节点端口需求
- [ ] 错误路径表（5 种）全部有清晰错误消息
- [ ] parser.py `from topology_collection import find_topology_file` 可正常 import（统一 PYTHONPATH / sys.path 过渡，无 importlib 临时代码）
- [ ] `_load_index` 按 P0-2 决策校验 `tgfw-topo-collection` 条目 + `compatible_consumers`（不要求 node* 顶层条目）
- [ ] 12 个单元测试全部通过（覆盖正常路径/错误路径/边界条件，含 `test_all_12_topos` / `test_list_topologies_returns_12` / `test_data_shape_distribution`）
- [ ] YAML 安全解析使用 `yaml.safe_load`
- [ ] 缓存机制正确（mtime 感知）
- [ ] `list_topologies` 返回 `_parse_collection_md` 解析出的全部 topo_id（12 个 node*，S1-A），与 index.yaml 顶层无关
- [ ] §5 示例与实际 collection.md 字段分布一致（S1-B：仅 `node2_dut1_tg1_link2` 有 speed_class/media_type，其余为简写）
- [ ] 模糊搜索在编辑距离 ≤2 时提供候选建议，候选集来自 `_parse_collection_md` 解析出的 12 个 node* topo_id（与 index.yaml 顶层无关，R3）
