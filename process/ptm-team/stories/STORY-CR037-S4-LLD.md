---
doc_type: LLD
story_id: "CR037-S4"
story_slug: "topo-mapper-enhance"
cr_id: "CR-037"
tier: "A"
status: "draft"
created_at: "2026-08-05"
owner: "meta-dev"
source_hld: "process/changes/CR-037-HLD.md §4.4"
source_story: "process/stories/STORY-CR037-S4.md"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "skills/topo-config/src/topo_mapper.py 增强（per-link 约束 + hardware_platform + 端口兼容矩阵 + ledger 占用闭环 + 约束叠加优先级）+ mapping_validator.py R2 兼容 + physical_pool.py release 接口"
---

# STORY-CR037-S4-LLD：topo_mapper.py 增强 详细设计

> **Tier**: A（回溯匹配剪枝接入，改动核心映射引擎，H1 跨用例防冲突）
> **Shared Fragments**: `skills/topo-config/src/topo_mapper.py`（增强）、`skills/topo-config/src/physical_pool.py`（新增 release + from_pool_data）、`skills/topo-config/src/mapping_validator.py`（R2 兼容，P1-6）、`skills/topo-planning/src/limit_parser.py`（TopoLimit 输入契约，S2）、`skills/topo-planning/src/pool_merge.py`（S3 最终池数据 → PhysicalPool，S3-B）
> **Open Items**: 无

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-08-05 | meta-dev | 初稿（14 章节 full-lld，CP5 提交） |
| v1.1 | 2026-08-05 | meta-dev（CP5 NEEDS_REWORK 修订） | S4-A（BLOCKER）：§5.2 H4 叠加公式修正——用户 speed_class 优先，TE 系强制 `media_type=fiber`、丢弃逻辑 copper，不产出 `{TE, copper}`；S4-B（BLOCKER）：§6.3 `_port_meets_requirement` 的 `not strict` 软约束分支移至硬剪枝 `return False` 之前，include-at-least-one 不再硬砍 GE 链路；S3-B：物理池输入统一为 raw PhysicalPool（`PhysicalPool.from_pool_data`），S4 §0/§4/§5.4/§8.1 与 S3/S7 消费一致；S4-C：确认 ledger 台账 schema list+port_status（P0-3）不变；S4-D：确认 mapping_validator R2 per-link 聚合不静默跳过 |

---

## 0. 工程依据

本模块是 CR-037 topo-planning 链路的映射引擎增强。工程依据源自 HLD §4.4（topo_mapper 增强：per-link port_requirement + hardware_platform 字段 + 端口兼容矩阵 + ledger 占用闭环 H1 + 约束叠加优先级 H4），以及设计审查已关闭项 P1-6（`mapping_validator.py` R2 链路类型校验不得静默跳过）与 P1-7（释放接口归属明确：S4 定义 release，S8 挂接 ptm-te 编排 [7] 清理）。S4 消费 S2 `TopoLimit`（链路 3 模式 + 平台别名展开）作为约束输入；**物理池输入为 raw `PhysicalPool`**，由 S3 归并产出的最终池数据（`UnifiedPool.to_pool_data()` + `ledger_path`）经 `PhysicalPool.from_pool_data()` 构造（S3-B，评审 D 组契约），向 S5 exporter 输出带 per-link 需求记录的 `MappingResult`。核心价值：让「在海光3250上执行这个用例，其中需要包含 TE|10GE 链路」这类 per-link + 设备约束可被回溯引擎确定性求解，且跨用例占用通过台账 `allocated_to` 持久化，不重复占用同一物理端口。

---

## 1. 目标

增强 `skills/topo-config/src/topo_mapper.py`，实现：
1. `port_requirement` 从全局 dict 升级为 **per-link dict**（`{link_id: {speed_class, media_type}}`），回溯剪枝在 `_check_connectivity` / `_port_meets_requirement` 按 link_id 取约束（AG-2 推荐 A，提前剪枝）。
2. 物理池节点 `hardware_platform` 字段匹配（等价类语义，对齐 S2/S3：`HG3250`/`JL-HG`/`D2000` 等，P1-3）。
3. 端口兼容矩阵：GE=copper|fiber，TE/XTE/QTE=fiber-only；`_port_meets_requirement` 按矩阵判定端口是否满足（speed, media 组合）。
4. **ledger 占用闭环（H1，P0）**：`_is_port_busy` 消费台账 `allocated_to`（跨用例持久占用，现有行为保留）；映射成功后按 `env_name` 写回 `allocate`；新增释放接口 `PhysicalPool.release(env_name)`，释放归属明确为 S4 定义、S8 挂接 ptm-te 编排 [7]（环境删除/teardown 调用）。
5. **约束叠加优先级（H4，S4-A）**：用户 topo-limit 链路约束 > 逻辑 topo（collection）自带约束（link.media_type + 端点接口 speed_class，P2-2）> 默认（无约束）。当用户约束 `speed_class` 为 TE 系（TE/XTE/QTE/TTE）时，`media_type` 强制 `fiber`（**丢弃逻辑 copper**——TE/QTE/XTE 仅 fiber，copper 与 TE 互斥），不得产出不可满足的 `{TE, copper}`。
6. **mapping_validator 兼容（P1-6）**：`port_requirement` 改 per-link 后，`mapping_validator.py:134-135` R2 链路类型校验不得静默跳过；采用显式 per-link 聚合兼容方案，`mapping_validator.py` 纳入 S4 文件影响范围。

---

## 2. Requirements

### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | `match(logical_topo, topo_limit=None, port_requirement=None, env_name=None)`：`port_requirement` 兼容 3 种输入——None（无端口约束）、全局 dict `{speed_class, media_type}`（作用于全部 link，向后兼容）、per-link dict `{link_id: {speed_class, media_type}}` | P0 |
| FR2 | `topo_limit`（S2 `TopoLimit`）接入：`device_constraints` 过滤候选（node_type + min_ports + device_type + **device_types 列表** + **hardware_platform 等价类**）；`link_constraints` 3 模式（specific/all/include-at-least-one）与逻辑 topo 自带约束按 H4 叠加 | P0 |
| FR3 | 候选剪枝新增 `hardware_platform` 等价类匹配：`pool.nodes[].hardware_platform == constraint.equiv_class`（如 `HG3250`）；`device_types` 列表命中 `pool.nodes[].device_type` 也满足 | P0 |
| FR4 | 端口兼容矩阵校验（内联常量，与 S2 `PORT_COMPAT_MATRIX` 同源）：GE→{copper,fiber}，TE/XTE/QTE/TTE→{fiber}，FE→{copper,fiber}；`_port_meets_requirement` 使用矩阵判定（requested speed/media vs 端口 speed/media） | P0 |
| FR5 | ledger 占用闭环（H1）：映射成功且 `env_name` 非 None 时，对本映射全部占用端口执行 `pool.allocate(device_name, port_id, env_name, vlan)` + `pool.save_ledger(ledger_path)`；`_is_port_busy` 继续消费台账 `allocated_to`（跨用例持久占用） | P0 |
| FR6 | 释放接口 `PhysicalPool.release(env_name)`：释放该 env_name 全部端口占用（`allocated_to`/`allocated_vlan` 置空），并派生设备态（无剩余占用 → free）；释放语义/时序由 S4 定义，调用挂接由 S8 在 ptm-te 编排 [7] 清理时执行 | P0 |
| FR7 | 约束叠加（H4，S4-A）：用户 topo-limit 链路约束（specific/all）> 逻辑 topo 自带（link.media_type + 端点接口 speed_class，P2-2）> 默认；用户 `speed_class` 优先，TE 系（TE/XTE/QTE/TTE）强制 `media_type=fiber`（丢弃逻辑 copper），逻辑 `media_type` 仅在无用户 speed_class 约束时生效 | P0 |
| FR8 | `include-at-least-one` 模式（S4-B）：回溯期按软约束（strict=False）处理——`not strict` 分支必须位于硬剪枝 `return False` 之前，其余 GE 链路不因「非 TE」被硬砍；post-match 校验 ≥1 条满足则成功，0 条满足则映射 failed（结构化报告「缺 TE 端口」类明细） | P0 |
| FR9 | mapping_validator 兼容（P1-6）：`MappingValidator.validate` 支持 per-link `port_requirement`，R2 按 `PortMapping.via_link` 聚合 per-link 需求；有约束时逐条校验、不得静默跳过 | P0 |
| FR10 | `MappingResult` 新增 `per_link_requirements: Dict[str, dict]` 与 `allocated_env: Optional[str]`，供 S5 exporter 与 S10 验证消费 | P1 |
| FR11 | 空 limit / 空 port_requirement：`topo_limit=None` 且 `port_requirement=None` 时保持现有行为（仅按 node_type + min_ports 匹配），向后兼容 | P0 |

### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 向后兼容：`match(topo)`（无新参数）行为与改造前等价，现有 `test_topo_mapper.py` 用例不回归 |
| NFR2 | 台账 schema 不变（P0-3）：`list + port_status[port].{allocated_to, allocated_vlan}`；`physical_pool.py` 既有 `_build_index`/`get_free_ports`/`allocate`/`save_ledger` 不因 S4 改变 |
| NFR3 | 无凭据暴露：`allocate` 写回只含 `allocated_to`/`allocated_vlan`，不透传密码 |
| NFR4 | 确定性：同输入同池同台账 → 同映射结果（候选遍历顺序稳定） |
| NFR5 | 性能：per-link 剪枝复杂度可控；全局约束场景退化为现有路径（HLD §9 回退策略） |

---

## 3. 模块拆分与职责

不拆子模块。`topo_mapper.py` 内部新增/修改以下函数：

| 函数 | 类型 | 职责 |
|------|------|------|
| `match(logical_topo, topo_limit=None, port_requirement=None, env_name=None)` | 主入口（改） | 串联 per-link 需求解析 + 候选构建（含 hardware_platform）+ 回溯剪枝 + 结果构建 + 可选 ledger 写回 |
| `_build_candidates(logical_topo)` | 修改 | 新增 `hardware_platform` 等价类 + `device_types` 列表过滤 |
| `_resolve_per_link_requirement(link_id)` | 新增 | H4 叠加（S4-A）：逻辑 topo 自带 + 用户约束 → 有效 per-link 需求（TE 系强制 fiber，丢弃逻辑 copper）+ strict 标志 |
| `_logical_topo_link_requirement(link_id)` | 新增 | 从 `links[lid].media_type` + 端点接口 `speed_class`（P2-2）推导逻辑自带约束 |
| `_user_link_requirement(link_id)` | 新增 | 从 `topo_limit.link_constraints` 解析用户约束（specific/all 绑定；include-at-least-one 走 soft，strict=False，S4-B） |
| `_check_connectivity(...)` | 修改 | 对每条 link_id 取 `_resolve_per_link_requirement`，传入 `_port_meets_requirement` |
| `_port_meets_requirement(device_name, port_id, requirement, strict=True)` | 修改 | 按端口兼容矩阵判定；include-at-least-one 时 strict=False（soft） |
| `_try_l2_pass_through(...)` | 修改 | 端点端口按 per-link 需求校验；SW 透传端口不校验类型（现有语义保留） |
| `_commit_allocation(assignment, used_ports, env_name)` | 新增 | 映射成功后写回台账（allocate + save_ledger） |
| `release_env(env_name)` | 新增（薄封装） | 调用 `self.pool.release(env_name)` + `save_ledger`，供外部（S8）调用 |
| `_build_result(...)` | 修改 | 回填 `per_link_requirements` 与 `allocated_env` |

`physical_pool.py` 新增方法：

| 方法 | 职责 |
|------|------|
| `release(env_name)` | 释放指定 env_name 的全部端口占用 + 设备态派生（free）；**台账 schema 不变**（P0-3） |
| `from_pool_data(pool_data, ledger_path)` | 从 S3 归并产出的最终池数据构造 `PhysicalPool`（S3-B，评审 D 组契约）；台账仍从 `ledger_path` 加载；**台账 schema 不变**（P0-3） |

`mapping_validator.py` 修改：

| 方法 | 职责 |
|------|------|
| `validate(mapping_result, logical_topo, topo_limit=None, port_requirement=None)` | 主入口增加 per-link 兼容；R2 不静默跳过（P1-6） |
| `_r2_link_type(...)` | 按 `pm.via_link` 解析 per-link 需求，逐条校验 |

---

## 4. 代码结构与文件影响范围

### 修改文件（S4 写入范围）

```
skills/topo-config/src/topo_mapper.py          # 增强：per-link + hardware_platform + 矩阵 + 写回
skills/topo-config/src/mapping_validator.py     # R2 兼容（P1-6）：per-link 聚合，不静默跳过
skills/topo-config/src/physical_pool.py         # 新增 release(env_name) + from_pool_data()（仅新增方法，台账 schema 不变，P0-3/P1-7/S3-B）
skills/topo-config/src/topology_model.py        # 可选：MappingResult 增加 per_link_requirements/allocated_env 字段
```

### 只读消费（不修改）

```
skills/topo-planning/src/limit_parser.py        # TopoLimit 数据类（S2 输出，只读）
skills/topo-planning/src/pool_merge.py          # S3 merge_pool → UnifiedPool（最终池数据，只读）；S4 用 to_pool_data() 构造 PhysicalPool（S3-B）
skills/topo-config/config/physical_ledger.yaml  # list + port_status schema（P0-3，不修改）
```

### 文件所有权与范围说明

- `topo_mapper.py`：S4 primary owner（`topo_mapper.py#per-link 约束 + ledger 闭环`）。
- `mapping_validator.py`：纳入 S4 范围（P1-6 明确要求），所有权随 S4；无其他 Wave 2 Story 占用。
- `physical_pool.py`：**仅新增 `release(env_name)` 与 `from_pool_data()` 两个方法**。约束「physical_pool.py 不修改」指**台账 schema 与既有方法不因 P0-3 改变**；`release` 是 H1/P1-7 要求的加性接口，`from_pool_data` 是 S3-B 要求的加性构造器（S3 最终池数据 → PhysicalPool），均不改变 schema、不改既有方法体。
- `topology_model.py`：DEVELOPMENT-PLAN 将 S4 output_files 列为 `topo_mapper.py + topology_model.py`；实际仅需在 `MappingResult` 增加两个字段，无结构性改动。

---

## 5. 数据模型与持久化设计

### 5.1 per-link port_requirement 结构

```python
# match() 内部归一化为统一 per-link 结构：
{
  "link1": {"speed_class": "TE", "media_type": "fiber"},
  "link2": {"speed_class": "GE", "media_type": "copper"},
  # 无约束的 link 不出现或为空 dict
}

# 兼容输入归一化规则：
#   None                                        → {}（全部无约束，向后兼容）
#   {"speed_class": "TE", "media_type": "fiber"}（全局）→ 应用到全部 link_id
#   {"link1": {...}, "link2": {...}}（per-link）       → 原样使用
```

### 5.2 约束叠加（H4）解析结果

`_resolve_per_link_requirement(link_id)` 返回 `(dict, strict)`（有效需求 + 是否硬约束）：

```
C_eff = {}
C_base = _logical_topo_link_requirement(link_id)   # 逻辑 topo 自带（下限）
C_user = _user_link_requirement(link_id)           # 用户约束（覆盖层，含 strict 标志）
C_eff.speed_class = C_user.speed_class or C_base.speed_class
# S4-A 修正：用户 speed_class 优先；TE 系（TE/XTE/QTE/TTE）强制 media_type=fiber，
# 丢弃逻辑 copper（copper 与 TE 互斥），逻辑 media_type 仅在无用户 speed_class 约束时生效。
if C_eff.speed_class in ('TE', 'XTE', 'QTE', 'TTE'):
    C_eff.media_type = 'fiber'
else:
    C_eff.media_type = C_user.media_type or C_base.media_type
```

**strict 标志（S4-B）**：`_user_link_requirement` 对 `include-at-least-one` 返回 `(req, strict=False)`（软约束，回溯不剪枝）；对 `specific`/`all` 返回 `(req, strict=True)`；逻辑自带约束恒为 `strict=True`。`_resolve_per_link_requirement` 返回 `(C_eff, strict)`，供 `_port_meets_requirement(dev, port, req, strict)` 判定。

示例（H4，HLD §4.4）：collection link 声明 `media_type: copper`（逻辑下限），用户 limit `speed_class: TE`（覆盖）→ **有效需求 `{speed_class: TE, media_type: fiber}`**（S4-A 丢弃逻辑 copper），不会产出不可满足的 `{TE, copper}`；TE-fiber 端口可满足。

### 5.3 MappingResult 扩展

```python
@dataclass
class MappingResult:
    ...
    per_link_requirements: Dict[str, dict] = field(default_factory=dict)  # link_id → 有效需求（供 S5/validator）
    allocated_env: Optional[str] = None            # 本次映射写回的 env_name（None=未写回）
```

### 5.4 持久化

- **池来源（S3-B）**：`TopoMapper` 接收的 `pool: PhysicalPool` 由 S3 最终池数据经 `PhysicalPool.from_pool_data(unified.to_pool_data(), unified.ledger_path)` 构造（不再直接 `PhysicalPool.load` 原始 pool 文件），保证 devices 覆盖/三角映射后的 `device_type`/`hardware_platform` 对 S4 可见。
- **台账写入**：映射成功 + `env_name` 非 None → 对 `used_ports` 每个 `(device, port_id)` 调 `pool.allocate(device, port_id, env_name, vlan=None)`，随后 `pool.save_ledger(self.pool.ledger_path)`（原子写，`physical_pool.py:279` 现有实现）。
- **台账释放**：`PhysicalPool.release(env_name)` 仅操作内存台账，随后由调用方 `save_ledger` 持久化。
- 台账 schema **保持 `list + port_status[port].{allocated_to, allocated_vlan}`**（P0-3，S4-C），`allocate`/`release`/`from_pool_data` 均基于该 schema。

---

## 6. API / Interface 设计

### 6.1 `TopoMapper.match(logical_topo, topo_limit=None, port_requirement=None, env_name=None) -> MappingResult`

**主入口**。参数：
- `logical_topo`：`Topology` 对象（S1 parser 输出）
- `topo_limit`：`TopoLimit`（S2），可选。含 `device_constraints` + `link_constraints`。
- `port_requirement`：None | 全局 dict | per-link dict（见 §5.1 归一化）
- `env_name`：可选 str。非 None 且映射成功 → 写回台账占用。

**归一化流程**：
1. `user_per_link = _normalize_port_requirement(port_requirement)`（全局 → 全部 link）
2. `self.topo_limit = topo_limit`
3. 候选构建 `_build_candidates`（含 hardware_platform/device_types 过滤）
4. 回溯（per-link 需求经 `_resolve_per_link_requirement` 注入 `_check_connectivity`）
5. 结果构建 + `per_link_requirements` 回填
6. 若 `env_name` 且结果 `status in (matched, partial)` → `_commit_allocation`

**异常**：无新异常类型。映射失败以 `MappingResult.status` + `reason` 表达（结构化，不抛裸异常）。

### 6.2 `_check_connectivity(logical_id, physical_name, assignment, used_ports) -> Optional[Tuple[set, dict]]`

按 link_id 取 `_resolve_per_link_requirement(lid)`（返回 `(req, strict)`，S4-B）；直连 cable 与 L2 透传端点均调用 `_port_meets_requirement(dev, port, req, strict)`（strict 按约束来源：specific/all/逻辑自带=True，include-at-least-one=False）。返回 `(ports_needed, auto_insertions)` 或 None。

### 6.3 `_port_meets_requirement(device_name, port_id, requirement, strict=True) -> bool`

端口兼容矩阵判定（FR4）：

```python
PORT_COMPAT_MATRIX = {
    'GE':  {'copper', 'fiber'},
    'TE':  {'fiber'},
    'XTE': {'fiber'},
    'QTE': {'fiber'},
    'TTE': {'fiber'},
    'FE':  {'copper', 'fiber'},
}

def _port_meets_requirement(self, device_name, port_id, requirement, strict=True):
    if not requirement:
        return True
    # S4-B 修正：软约束（include-at-least-one，strict=False）分支必须位于硬剪枝 return False 之前。
    # 软约束下其余 GE 链路不因「非 TE」被硬砍，由 post-match 计数 ≥1 判成功 / 0 条失败。
    if not strict:
        return True
    iface = self.pool.get_interfaces(device_name).get(port_id, {})
    port_speed = iface.get('speed_class')
    port_media = iface.get('media_type')
    req_speed = requirement.get('speed_class')
    req_media = requirement.get('media_type')
    if req_speed and port_speed != req_speed:
        return False
    if req_media:
        if port_media != req_media:
            return False
        allowed = PORT_COMPAT_MATRIX.get(port_speed, set())
        if port_speed and req_media not in allowed:
            return False
    return True
```

> 说明：`strict=False` 分支用于 `include-at-least-one`（S4-B）；`specific`/`all`/逻辑自带约束使用 `strict=True`。`{TE, copper}` 等非法组合不会出现——H4 合并阶段（§5.2）已对 TE 系强制 `media_type=fiber`（S4-A）。

### 6.4 `PhysicalPool.release(env_name: str) -> int`

新增方法（`physical_pool.py`）：

```python
def release(self, env_name):
    """释放指定 env_name 的全部端口占用（H1/P1-7）。

    台账 schema 不变（list + port_status，P0-3）。调用方随后 save_ledger 持久化。
    返回释放的端口数。
    """
    released = 0
    for dev in (self.ledger_data.get('devices') or []):
        ps = dev.get('port_status') or {}
        for port_id, entry in ps.items():
            if entry.get('allocated_to') == env_name:
                entry['allocated_to'] = None
                entry['allocated_vlan'] = None
                released += 1
        # 设备态派生：无剩余占用 → free（保持 faulty 不被覆盖）
        if released and all(p.get('allocated_to') is None for p in ps.values()):
            if dev.get('status') != 'faulty':
                dev['status'] = 'free'
    return released
```

**释放归属（P1-7）**：S4 定义接口 + 单测语义；S8 在 ptm-te 编排 [7] 清理流程（环境删除/teardown）挂接调用 `pool.release(env_name)` + `pool.save_ledger(ledger_path)`，避免只写不释放。

### 6.4b `PhysicalPool.from_pool_data(pool_data: dict, ledger_path: str) -> PhysicalPool`

新增 classmethod（S3-B，评审 D 组契约）：

```python
@classmethod
def from_pool_data(cls, pool_data, ledger_path):
    """从 S3 归并产出的最终池数据构造 PhysicalPool（S3-B，评审 D 组契约）。

    与 `load` 的区别：pool_data 直接来自内存（UnifiedPool.to_pool_data()），
    不重新读 physical_pool.yaml 原始文件，保证 devices 覆盖/三角映射后的
    device_type/hardware_platform 对 topo_mapper 可见；台账仍从 ledger_path 加载。
    台账 schema 不变（list + port_status，P0-3，S4-C）。
    """
    p = cls()
    if not isinstance(pool_data, dict):
        raise ValueError("pool_data must be a mapping")
    p.pool_data = pool_data
    p.ledger_path = ledger_path
    with open(ledger_path, 'r', encoding='utf-8') as f:
        p.ledger_data = yaml.safe_load(f)
    if not isinstance(p.ledger_data, dict):
        raise ValueError(f"Invalid YAML in ledger file '{ledger_path}'")
    p._build_index()
    return p
```

**构造路径**：`S3 merge_pool → UnifiedPool → to_pool_data() → PhysicalPool.from_pool_data()`；S7 CLI `_build_physical_view` 与 S4 集成/单测均走此路径，避免重新加载未归并的原始 pool 文件（S3-B 与 S7 §6.4 消费一致）。

### 6.5 `MappingValidator.validate(mapping_result, logical_topo, topo_limit=None, port_requirement=None) -> MappingValidationResult`

R2 兼容（P1-6/S4-D）：

- `port_requirement` 支持全局 dict 与 per-link dict（与 §5.1 归一化一致）。
- R2 对每个 `DeviceMapping.port_mappings[]`，按 `pm.via_link` 解析有效需求（复用与 topo_mapper 相同的 H4 解析函数，或读取 `MappingResult.per_link_requirements`）。
- 有约束的 link 逐端口校验；无约束的 link 计入 checked 并标注 `unconstrained`——**不静默跳过**（P1-6）。
- `include-at-least-one` 软约束在 R2 中按「**≥1 条 link 满足**」聚合校验（与 S4-B 语义一致），不做 per-link 全量硬判；0 条满足 → R2 失败 + 结构化明细。
- 若 `port_requirement`/`topo_limit` 均为 None → R2 返回「无约束，跳过」（与现状一致，逻辑层无接口属性）。

### 6.6 `TopoMapper.release_env(env_name) -> int`

薄封装：`n = self.pool.release(env_name)`；若 `self.pool.ledger_path` 存在则 `save_ledger`；返回释放端口数。供 S7 CLI / S8 编排调用。

---

## 7. 核心处理流程

### 7.1 主流程

```
TopoMapper.match(logical_topo, topo_limit, port_requirement, env_name)
 │
 ├─ 1. user_per_link = _normalize_port_requirement(port_requirement)
 │       # 全局 dict → 应用到全部 link；per-link dict → 原样；None → {}
 │
 ├─ 2. candidates = _build_candidates(logical_topo)
 │       # 对每个逻辑节点：
 │       #   Mock → []
 │       #   node_type/node_types 过滤（现有）
 │       #   min_ports 数量过滤（现有 get_free_ports 按空闲判定）
 │       #   node.device_type 精确匹配（现有）
 │       #   新增：topo_limit.device_constraints 匹配该逻辑节点时：
 │       #        device_types 列表命中 pool.device_type，或
 │       #        equiv_class 命中 pool.hardware_platform（等价类，P1-3）
 │
 ├─ 3. ordered = _order_logical_nodes(logical_topo)   # 现有排序
 │
 ├─ 4. ok = _backtrack(ordered, candidates, assignment, used_ports, 0, optional_pending)
 │       # 每候选调用 _check_connectivity(nid, cand, assignment, used_ports)
 │       #   → 对每条 link_id: req = _resolve_per_link_requirement(lid)
 │       #      直连 cable: 两端 _port_meets_requirement(dev, port, req, strict)
 │       #      L2 透传: 端点端口同样校验，SW 端口不校验类型
 │
 ├─ 5. result = _build_result(...)   # 回填 per_link_requirements + allocated_env
 │
 └─ 6. if env_name and result.status in ('matched', 'partial'):
          _commit_allocation(assignment, result, env_name)
          # for (dev, port) in used_ports: pool.allocate(dev, port, env_name, vlan=None)
          # pool.save_ledger(pool.ledger_path)
          # result.allocated_env = env_name
```

### 7.2 include-at-least-one 流程（FR8，S4-B）

```
1. _user_link_requirement(lid) 对 include-at-least-one 返回 (req, strict=False)。
2. 回溯期 _port_meets_requirement(..., strict=False) 直接返回 True（不剪枝）——
   软约束分支位于硬剪枝 return False 之前，其余 GE 链路不因「非 TE」被硬砍。
3. 回溯完成后 _build_result：
      satisfied = 统计 link_mappings 中满足该 include-at-least-one 需求的 matched link 数
      if satisfied >= 1: 成功（该约束满足）
      if satisfied == 0:
          result.status = 'failed'（若其余也失败）或 partial，
          reason 含「缺 TE 端口」类结构化明细（供 S5 失败报告）
```

### 7.3 ledger 占用闭环时序（H1，S4-C）

```
映射前（_is_port_busy）：used_ports（本次）∪ 台账 allocated_to 非空（跨用例）→ 端口 busy
映射成功（_commit_allocation）：
    pool.allocate(dev, port_id, env_name, vlan=None)  # 写内存台账
    pool.save_ledger(pool.ledger_path)                # 原子持久化
释放（S8 编排 [7] teardown）：
    pool.release(env_name)                            # 全部该 env 端口置空 + 设备态派生
    pool.save_ledger(pool.ledger_path)                # 持久化
```

**S4-C（台账 schema 不变，P0-3）**：`_is_port_busy` 消费 `allocated_to`、`allocate` 写回 `allocated_to`/`allocated_vlan`、`release` 置空，全部基于 `physical_ledger.yaml` 现有 **`list + port_status: {port: {allocated_to, allocated_vlan}}`** schema；不新增字段、不改结构。

### 错误路径

| 步骤 | 失败条件 | 行为 |
|------|---------|------|
| 2 | 逻辑节点无 node_type | `candidates[nid]=[]` + warning（现有） |
| 2 | hardware_platform 约束无候选 | 候选为空 → 该节点 unmapped + reason「无满足 hardware_platform=HG3250 的空闲设备」 |
| 4 | per-link TE 约束无 TE 端口 | 回溯失败 → 设备/link unmapped + reason「物理池无 TE 端口」 |
| 4 | include-at-least-one 0 条满足 | result.status=failed/partial + reason 明细 |
| 6 | env_name 写回时端口已被并发占用 | `allocate` 返回 False → warning + 不写该端口（台账是单写源，S8 串行编排） |

---

## 8. 技术细节

### 8.1 候选构建增强（hardware_platform / device_types）

> **池来源（S3-B）**：`self.pool` 为 raw `PhysicalPool`，由 S3 最终池数据（`UnifiedPool.to_pool_data()`）经 `PhysicalPool.from_pool_data()` 构造。pool 节点已含 `hardware_platform`（P-1-B 模板 + S3 devices 覆盖/三角映射），故 `dev.get('hardware_platform')` 在 S4 可见。

```python
def _build_candidates(self, logical_topo):
    candidates = {}
    for nid, node in logical_topo.nodes.items():
        if self._is_mock(node):
            candidates[nid] = []
            continue
        need_count = node.min_ports or 0
        types = node.node_types or ([node.node_type] if node.node_type else [])
        if not types:
            candidates[nid] = []
            continue
        # 本逻辑节点命中的设备约束（S2）
        dev_con = self._device_constraint_for(nid, node)   # 新增
        pool_devs = self.pool.get_devices_by_type(types)
        cands = []
        for dev in pool_devs:
            name = dev['name']
            if self.pool.is_faulty(name):
                continue
            free = self.pool.get_free_ports(name)
            if len(free) < need_count:
                continue
            if node.device_type and self.pool.get_device_type(name) != node.device_type:
                continue
            # ── S4 新增：设备约束过滤 ──
            if dev_con:
                if not self._device_meets_constraint(dev, dev_con):
                    continue
            cands.append(name)
        candidates[nid] = cands
    return candidates

def _device_meets_constraint(self, dev, con) -> bool:
    """设备约束命中判定（等价类语义，P1-3）：
       1) device_types 列表命中 pool.device_type；
       2) 或 equiv_class 命中 pool.hardware_platform；
       3) 冲突型号（device_type 在 conflicts 中）时，pool.hardware_platform 等价类匹配。
    """
    if con.get('device_types') and dev.get('device_type') in con['device_types']:
        return True
    if con.get('equiv_class'):
        if dev.get('hardware_platform') == con['equiv_class']:
            return True
        # 等价类兜底：若池未填 hardware_platform，回退 device_type 反查（S2/S3 语义）
        return False
    return True
```

### 8.2 逻辑 topo 自带约束推导（P2-2）

```python
def _logical_topo_link_requirement(self, link_id) -> dict:
    link = self.logical_topo.links.get(link_id)
    if not link:
        return {}
    req = {}
    # 端点接口 speed_class（collection 的 interfaces[].speed_class，P2-2）
    speeds = set()
    for ep in (link.endpoints or []):
        node = self.logical_topo.nodes.get(ep.node)
        if node and ep.interface and ep.interface in (node.interfaces or {}):
            s = node.interfaces[ep.interface].speed_class
            if s:
                speeds.add(s.upper())
    if len(speeds) == 1:
        req['speed_class'] = next(iter(speeds))
    elif len(speeds) > 1:
        # 端点速度不一致：取端口兼容矩阵中更严者（fiber-only 判定），缺省不设 speed
        pass
    # link.media_type
    if link.media_type and link.media_type.lower() in ('copper', 'fiber'):
        req['media_type'] = link.media_type.lower()
    return req
```

### 8.3 用户约束解析与 H4 叠加

```python
def _resolve_per_link_requirement(self, link_id) -> tuple:
    """H4 叠加（S4-A）：返回 (C_eff, strict)。
    C_eff = 用户字段优先 + 逻辑下限字段补齐；TE 系 speed_class 强制 media_type=fiber，
    丢弃逻辑 copper；逻辑 media_type 仅在无用户 speed_class 约束时生效。"""
    base = self._logical_topo_link_requirement(link_id)
    user_req, strict = self._user_link_requirement(link_id)
    eff = {}
    eff['speed_class'] = user_req.get('speed_class') or base.get('speed_class')
    if eff.get('speed_class') in ('TE', 'XTE', 'QTE', 'TTE'):
        eff['media_type'] = 'fiber'               # S4-A：copper 与 TE 互斥，丢弃逻辑 copper
    else:
        eff['media_type'] = user_req.get('media_type') or base.get('media_type')
    return eff, strict

def _user_link_requirement(self, link_id) -> tuple:
    """从 topo_limit.link_constraints 解析用户约束，返回 (req: dict, strict: bool)。
    S4-B：include-at-least-one → strict=False（软约束，不硬剪枝）；
    specific/all → strict=True（硬约束）。"""
    if not self.topo_limit:
        return {}, True
    for lc in self.topo_limit.link_constraints:
        mode = lc.mode
        if mode == 'specific' and lc.link_id == link_id:
            return self._constraint_dict(lc), True
        if mode == 'all':
            return self._constraint_dict(lc), True
        if mode == 'include-at-least-one':
            return self._constraint_dict(lc), False   # 软约束（S4-B）
    return {}, True

def _constraint_dict(self, lc) -> dict:
    d = {}
    if lc.speed_class: d['speed_class'] = lc.speed_class.upper()
    if lc.media_type:  d['media_type']  = lc.media_type.lower()
    return d
```

### 8.4 R2 per-link 聚合（mapping_validator.py，P1-6）

```python
def _r2_link_type(self, mapping, logical_topo):
    """链路类型一致：按 PortMapping.via_link 聚合 per-link 需求，逐端口校验。
    P1-6：port_requirement 改 per-link 后不得静默跳过——有约束的 link 全部校验，
    无约束的 link 计入 checked 并标注 unconstrained。"""
    if not self.port_requirement and not self.topo_limit:
        return CheckResult('R2:链路类型一致', True,
                           '无 port_requirement/topo_limit，跳过（逻辑层无接口属性约束）')

    failures = []
    checked = 0
    unconstrained = 0
    for dm in mapping.device_mappings:
        if getattr(dm, 'status', '') != 'matched':
            continue
        for pm in dm.port_mappings:
            link_id = getattr(pm, 'via_link', None)
            req = self._resolve_requirement_for_link(link_id)   # 复用 H4 解析或 MappingResult.per_link_requirements
            if not req:
                unconstrained += 1
                continue
            checked += 1
            phys_iface = self.pool.get_interfaces(dm.physical).get(pm.physical_port_id, {})
            if req.get('speed_class') and phys_iface.get('speed_class') != req['speed_class']:
                failures.append({...})
            if req.get('media_type') and phys_iface.get('media_type') != req['media_type']:
                failures.append({...})

    if failures:
        return CheckResult('R2:链路类型一致', False, f'{len(failures)} 个端口类型不满足 per-link 约束', failures)
    note = f'{checked} 个端口满足 per-link 约束' + (f'，{unconstrained} 个端口无约束' if unconstrained else '')
    return CheckResult('R2:链路类型一致', True, note)
```

> 说明（S4-D/S4-B）：`include-at-least-one` 软约束在 R2 中按「≥1 条 link 满足」聚合判定，不做 per-link 全量硬判；`_resolve_requirement_for_link` 对软约束返回的需求仅用于计数，不产生 per-port 失败。

---

## 9. 安全与性能设计

### 安全

- **无凭据暴露**：`allocate`/`release` 只操作 `allocated_to`/`allocated_vlan`，不透传密码/Token。
- **台账单写**：映射写回与释放均为内存台账 + `save_ledger` 原子写（`os.replace`）；S8 编排串行调用，避免并发写冲突。
- **不越权**：S4 不修改 `physical_ledger.yaml` schema（P0-3），不触碰凭据字段。

### 性能

- per-link 需求解析为纯内存 dict 查表（每 link O(1)）。
- 候选剪枝新增 `hardware_platform` 过滤为 O(候选数)，与现有 `get_free_ports` 同级。
- 全局约束退化路径（`port_requirement` 全局 dict 应用全部 link）与现状等价（HLD §9 回退策略）。
- 台账写回仅发生在映射成功（matched/partial）时，单次 `save_ledger` 原子写。

---

## 10. 测试设计

### 单元测试（`skills/topo-config/tests/test_topo_mapper.py` 扩展 + `tests/test_mapping_validator.py` 扩展）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_per_link_te_constraint_matches` | fixture 池含 TE 端口；`port_requirement={'link1': {'speed_class': 'TE'}}` | 映射成功，link1 端口 speed_class=TE |
| `test_per_link_te_constraint_no_te` | fixture 池无 TE 端口；TE per-link 约束 | 映射 failed + reason「缺 TE 端口」 |
| `test_global_requirement_backward_compat` | `port_requirement={'speed_class': 'GE'}`（全局） | 应用全部 link，映射成功 |
| `test_hardware_platform_equiv_class` | `topo_limit` device_constraint `equiv_class=HG3250` | 仅 `hardware_platform=HG3250` 的设备入选 |
| `test_hardware_platform_no_candidate` | `equiv_class=HG5380` 无匹配 | 节点 unmapped + reason |
| `test_device_types_list` | `device_types=[DAS-TGFW-A1300-HU]` | 仅该型号入选 |
| `test_port_compat_matrix_ge_fiber` | 需求 `{speed_class: GE, media_type: fiber}` | GE-fiber 端口满足；GE-copper 不满足 |
| `test_port_compat_matrix_te_fiber_only` | 需求 `{speed_class: TE, media_type: fiber}` | TE 端口满足 |
| `test_h4_user_overrides_logic` | 逻辑 link 自带 `media_type: copper`（下限）+ 用户 `speed_class: TE`（覆盖） | 有效需求 `{TE, fiber}`（S4-A 丢弃逻辑 copper），映射按 TE-fiber 求解 |
| `test_h4_logic_floor_applies` | 逻辑 link `media_type: fiber`，无用户约束 | 有效需求 media_type=fiber，仅 fiber 端口入选 |
| `test_include_at_least_one_satisfied` | include-at-least-one TE；池有 1 条 TE link + 其余 GE | 映射成功，≥1 条 TE link |
| `test_include_at_least_one_soft_not_hardcut` | include-at-least-one TE；部分 GE 链路非 TE | 软约束不硬砍 GE（S4-B），post-match ≥1 则成功 |
| `test_include_at_least_one_zero` | include-at-least-one TE；池无 TE | 0 条满足 → failed + 结构化明细 |
| `test_from_pool_data_construction` | `PhysicalPool.from_pool_data(unified.to_pool_data(), ledger_path)`（S3-B） | 构造成功，`get_devices_by_type` 返回节点含 `hardware_platform` |
| `test_ledger_allocate_on_success` | `env_name='case-001'` 映射成功 | 台账 `port_status[port].allocated_to='case-001'`，`save_ledger` 落盘 |
| `test_ledger_release` | `pool.release('case-001')` | 该 env 全部端口置空，设备态 → free |
| `test_ledger_cross_use_busy` | 端口已被 `allocated_to=case-000` 占用 | `_is_port_busy`=True，二次映射避开该端口 |
| `test_ledger_release_persist` | release 后 save_ledger → 重新 load | 端口恢复 free |
| `test_empty_limit_backward_compat` | `match(topo)` 无新参数 | 行为与改造前等价 |
| `test_mapping_result_per_link_requirements` | 映射结果含 per-link 有效需求 | `MappingResult.per_link_requirements` 正确 |
| `test_validator_r2_per_link` | `validate(..., port_requirement=per_link)` | R2 逐端口校验，不静默跳过（P1-6） |
| `test_validator_r2_unconstrained_recorded` | 部分 link 无约束 | checked 含 unconstrained 标注，不报假失败 |

### Fixture 需求

- `skills/topo-config/tests/fixtures/physical_pool.sample.yaml`（P-1 造数）：含 hg3250-51/52（`hardware_platform: HG3250`）+ TE 端口（hg3250-51 port3/4）。
- 新增 `tests/fixtures/ledger_allocated.sample.yaml`：预置 `allocated_to: case-000` 的台账，供跨用例占用测试。
- 新增 `tests/fixtures/pool_no_te.sample.yaml`：无 TE 端口的池，供失败路径测试。

### 集成测试

- S10：`S1 find_topology_file → parser → Topology` → `S2 TopoLimit` → `S3 merge_pool → UnifiedPool` → `S4 match` → 台账占用闭环（fixture 池）。
- 参考用例 `IPv4策略路由`（TOPO-01）：映射 → ledger 写回 → 二次映射避开 → release 恢复。

---

## 11. 实施步骤

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-S4-01 | `MappingResult` 增加 `per_link_requirements`/`allocated_env` 字段 + `topology_model.py` | 30min | 数据模型 |
| TASK-S4-02 | `_normalize_port_requirement` 归一化（None/全局/per-link） | 30min | 输入归一化 |
| TASK-S4-03 | `_resolve_per_link_requirement` + `_logical_topo_link_requirement` + `_user_link_requirement`（H4） | 1h | 约束叠加 |
| TASK-S4-04 | `_port_meets_requirement` 端口兼容矩阵改造（strict 参数） | 1h | 端口判定 |
| TASK-S4-05 | `_check_connectivity`/`_try_l2_pass_through` per-link 剪枝接入 | 1.5h | 回溯剪枝 |
| TASK-S4-06 | `_build_candidates` hardware_platform/device_types 过滤 | 1h | 候选过滤 |
| TASK-S4-07 | `_commit_allocation` + `env_name` 写回（allocate + save_ledger） | 1h | ledger 写回 |
| TASK-S4-08 | `PhysicalPool.release(env_name)` + `PhysicalPool.from_pool_data()`（S3-B）+ `TopoMapper.release_env` | 1h | 释放/构造接口（P1-7/S3-B） |
| TASK-S4-09 | include-at-least-one post-match 校验（FR8） | 1h | soft 约束 |
| TASK-S4-10 | `mapping_validator.py` R2 per-link 聚合（P1-6） | 1.5h | validator 兼容 |
| TASK-S4-11 | 单元测试（20 用例 + 3 fixture） | 2h | 测试 |
| **合计** | | **~12h** | |

---

## 12. 风险、难点与预研建议

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| per-link 剪枝增加回溯复杂度 | 中 | 全局约束退化为现有路径（HLD §9）；候选按 hardware_platform 预过滤缩小空间 |
| 端口兼容矩阵与 S2 矩阵不同步 | 中 | 单一常量定义（S2/S4 共用 `PORT_COMPAT_MATRIX` 语义），S10 测试校验一致 |
| 台账并发写冲突（S8 串行 vs 未来并行） | 低 | 台账单写源 + `save_ledger` 原子写；S8 编排串行 |
| `include-at-least-one` 回溯可能过早失败 | 中 | 软约束设计（strict=False 分支位于硬剪枝前，S4-B）+ post-match 计数；仍失败时输出结构化报告（预期行为） |
| `physical_pool.py` 增加 release/from_pool_data 超出原 output_files 预期 | 低 | 加性方法、schema 不变（P0-3）；文件所有权在 CP5 声明 S4 含 `physical_pool.py#release + #from_pool_data`（P1-7/S3-B） |
| 池节点未填 `hardware_platform` 导致等价类匹配失效 | 中 | S3 覆盖/台账三角映射补齐（S3-B）；P-1-B 模板必填；缺失时 warning + 按现有路径匹配 |

**预研建议**：
1. 用 fixture 池对「TE 存在/不存在」两种场景跑 dry-run，确认失败报告信息可用。
2. 与 S5 exporter 对齐 `MappingResult.per_link_requirements` 输出结构，避免转换层。

---

## 13. 实现灰区与取舍记录

### 灰区 1：`include-at-least-one` 回溯策略

**问题**：用户「至少一条 TE 链路」在回溯中如何处理，才能既不错杀（无 TE 时也应失败）也不过度约束（有 1 条 TE 即可）？

**选项**：
- A. 硬约束作用全部 link（回溯期每条 link 都要求 TE）——实现简单但池中仅 1 条 TE 时误失败。
- B. 软约束（strict=False）+ post-match 计数 ≥1——回溯不阻断，最终校验 ≥1，正确但需 post-match 兜底。
- C. 两阶段：先全硬，失败再放宽——复杂度高。

**决策**：选 B（修正 S4-B）。回溯期 `_port_meets_requirement(strict=False)` 的软约束分支**必须位于硬剪枝 `return False` 之前**——否则「非 TE」端口在 soft 语义下也会被硬砍，与软约束初衷矛盾（死代码）。修正后 post-match 统计满足 link 数，≥1 成功，0 条则 failed + 结构化明细（缺 TE 端口）。此方案正确且实现可控，符合 HLD「映射失败输出结构化限制校验报告，不兜底」。

### 灰区 2：H4 叠加中「用户覆盖 vs 逻辑下限」的冲突消解

**问题**：collection link 声明 `media_type: copper`，用户 limit 声明 `speed_class: TE`（TE=fiber-only），两约束冲突时取哪个？

**选项**：
- A. 用户完全覆盖（丢弃逻辑下限）——可能违背物理约束（copper 下限被忽略）。
- B. 取并集（更严者）：媒体类型取 fiber（TE 强制）、速度取 TE——满足 HLD「逻辑自带为下限、用户可加 TE 上限」。
- C. 报冲突错误。

**决策**：选 B（修正 S4-A）。有效需求 = 用户字段优先 + 逻辑下限字段补齐；但 **TE 系 speed_class 强制 `media_type=fiber`，丢弃逻辑 copper**（TE/QTE/XTE 仅 fiber，copper 与 TE 互斥），逻辑 `media_type` 仅在无用户 speed_class 约束时生效。修正前「TE + copper」组合虽会被矩阵判非法，但直接产出不可满足需求导致映射必然失败；修正后产出 `{TE, fiber}`，映射可求解。此语义写入 §5.2 与单测 `test_h4_user_overrides_logic`。

### 灰区 3：`PhysicalPool.release` 归属（P1-7）

**问题**：释放接口放哪、谁调用？

**选项**：
- A. S4 在 `physical_pool.py` 新增 `release(env_name)`，S8 挂接 ptm-te 编排 [7] 清理（环境删除/teardown）。
- B. 释放逻辑内联在 topo_mapper.py，不新增 pool 方法。
- C. 释放交给用户手动改台账。

**决策**：选 A。`release` 是物理池台账生命周期方法，与 `allocate` 对称，归属 `physical_pool.py` 最自然；调用时机由 S8 编排保证「环境删除/teardown 调用」，避免只写不释放。`physical_pool.py` 仅新增方法、不改 schema/既有方法（满足 P0-3 约束）。

### 灰区 4：mapping_validator R2 兼容方案（P1-6）

**问题**：`port_requirement` 改 per-link 后，R2 如何避免静默跳过？

**选项**：
- A. 保留全局语义归一化：validate 内部把全局 dict 展开为 per-link，再聚合校验。
- B. 显式 per-link 聚合：R2 按 `PortMapping.via_link` 逐端口解析需求，无约束端口标注 unconstrained。
- C. 从 MappingResult.per_link_requirements 直接读取（S4 已回填）。

**决策**：选 B + C 结合。R2 优先读取 `MappingResult.per_link_requirements`（S4 已按 H4 解析），缺失时按传入 `port_requirement`/`topo_limit` 复算；有约束端口全部校验，无约束端口计数并标注，**不静默跳过**。

---

## 14. 回滚与发布策略

### 回滚

- `topo_mapper.py`/`mapping_validator.py`/`physical_pool.py`/`topology_model.py` 变更可整体回退到改造前 commit；`release` 方法为加性，回退无副作用。
- 台账写回为运行时行为，回滚代码后台账 `allocated_to` 仍由 `release(env_name)` 清理（或人工置空）。
- 测试与 fixture 一并回退。

### 发布

- S4 随 topo-config 引擎发布，供 S5/S7/S8 消费；S8 安装时 `release` 挂接编排 [7]。
- 向后兼容：`match(topo)` 旧签名行为不变，现有调用方（EnvironmentManager/CLI）无感。
- 依赖顺序：S4 依赖 S1（逻辑 topo）/S2（TopoLimit）/S3（UnifiedPool）设计证据确认后方可实现。

---

## 15. Definition of Done（DoD）

- [ ] `port_requirement` 支持 None/全局/per-link 三种输入，归一化正确（FR1）
- [ ] `topo_limit` device_constraints 按 hardware_platform 等价类 + device_types 列表过滤候选（FR2/FR3，P1-3）
- [ ] 端口兼容矩阵判定正确：GE=copper|fiber，TE/XTE/QTE=fiber-only（FR4）
- [ ] ledger 占用闭环（H1）：映射成功写回 `allocated_to`；二次映射避开已占用端口；`release(env_name)` 释放后恢复 free（FR5/FR6）
- [ ] 约束叠加（H4，S4-A）：用户 > 逻辑自带（media_type + 端点 speed_class，P2-2）> 默认，单测覆盖；**TE 系强制 `media_type=fiber`，丢弃逻辑 copper，不产出 `{TE, copper}`**（FR7）
- [ ] include-at-least-one（S4-B）：**软约束分支位于硬剪枝 `return False` 之前**，其余 GE 链路不硬砍；≥1 条满足通过；0 条满足失败 + 结构化明细（FR8）
- [ ] mapping_validator R2 per-link 聚合，不静默跳过（FR9，P1-6/S4-D）
- [ ] `MappingResult.per_link_requirements` + `allocated_env` 回填（FR10）
- [ ] 空 limit 向后兼容：`match(topo)` 行为与改造前等价（FR11/NFR1）
- [ ] `physical_pool.py` 台账 schema 不变（P0-3，S4-C）；`release` 为加性方法（P1-7）；`from_pool_data` 为 S3-B 加性构造器（S3 最终池数据 → raw PhysicalPool）
- [ ] 20 个单元测试全部通过，现有 `test_topo_mapper.py`/`test_mapping_validator.py` 不回归
