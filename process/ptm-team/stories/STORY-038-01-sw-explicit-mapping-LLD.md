---
doc_type: LLD
story_id: STORY-038-01
story_slug: sw-explicit-mapping
cr_id: CR-038
feature_id: F-CR038-E1
title: SW 节点显式映射（含显式 SW 优先级 + role 保留）— LLD
design_evidence_type: full-lld
lld_policy_required_level: full-lld
tier: S
risk_level: medium
batch_scope: n/a
homogeneous_story_pattern: n/a
shared_contract: n/a
source_hld: docs/design/HLD-CR-038.md §10.1
source_adr: docs/design/ARCHITECTURE-DECISION-CR-038.md §ADR-CR038-01
source_feature: docs/features/cr038-sw-mapping/DESIGN.md
open_items: []
status: draft
version: "0.1"
created_at: "2026-08-15"
owner: meta-dev
---

# STORY-038-01 LLD — SW 节点显式映射（显式 SW 优先级 + role 保留）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-dev | 初稿：explicit_sw_reserved 预计算 + 透传跳过 + role 传递验证 + 结构化 reason + R-F-018 互斥单测设计 |

---

## 0. 工程依据

本 Story 是 F-CR038-E1（SW 映射与台账）的核心引擎改动，突破 CR-037「TG+DUT 直连」MVP，引入 SW 显式逻辑节点映射与自动 L2 透传的互斥。

- **HLD §10.1**：`topo_mapper` 确定性匹配算法，显式 SW 节点匹配到物理 SW 设备，`status=matched`。
- **ADR-CR038-01**：显式 SW reserved 集互斥（`explicit_sw_reserved`），保证显式 SW 优先于自动 L2 透传，与回溯顺序无关。
- **Feature DESIGN（cr038-sw-mapping）**：确定性算法、role 传递、台账互斥。
- **REQUIREMENTS**：R-F-001（SW 显式映射）、R-F-002（role 保留）、R-F-018（显式 SW 协调优先级）。

**文档结构导览（14 段语义要点对齐）**：§1 目标与需求，§2 文件影响范围（代码结构），§3 接口设计（内部 API），§4 数据模型，§5 核心流程与技术细节，§6 异常处理，§7 测试设计，§8 实施步骤，§9 回滚策略，§10 安全约束，§11 需求契约映射，§12 依赖与前置，§13 风险与开放项，§14 验收与交接（DoD）。

---

## 1. 背景与目标

### 1.1 Goal

在 `skills/topo-config/src/topo_mapper.py` 上增量实现三条能力，兑现 HLD-CR-038 §10.1：

1. **显式 SW 匹配**：`node_type=SW`（或 `node_types` 含 `SW`）的逻辑节点可显式匹配到物理 SW 设备，产出 `DeviceMapping.status='matched'`。
2. **显式 SW 优先（R-F-018）**：以 `explicit_sw_reserved` 集互斥，保证显式 SW 的物理候选永不落入自动 L2 透传候选池，且该互斥与回溯顺序无关。
3. **role 保留（R-F-002）**：`DeviceMapping.role` 保持传递逻辑节点 `node.role`（如 `pppoe-server`），不被自动透传覆盖。

### 1.2 Functional Requirements（本 Story 范围）

| 编号 | 需求 | 量化验收 |
|------|------|---------|
| R-F-001 | SW 显式节点映射 | 逻辑 SW 节点在 pool 有候选时 `DeviceMapping.status=matched`，成功率 100%（G1） |
| R-F-002 | role 保留 | 逻辑 SW 节点 `role=pppoe-server` → `DeviceMapping.role='pppoe-server'` |
| R-F-018 | 显式 SW 优先于自动透传 | `explicit_sw_reserved ∩ 自动透传候选 = ∅`，且交换节点处理顺序不变量不变 |
| R-NF-002 | 结构化失败 reason | SW 无候选 / 接口不匹配 → `DeviceMapping.reason` 非空且可区分两类失败 |

### 1.3 Non-Functional Requirements

| 编号 | 约束 | 设计响应 |
|------|------|---------|
| R-NF-004 | 回归无破坏 | `explicit_sw_reserved` 默认空集，node2 系列（无显式 SW）行为不变 |
| R-NF-003 | 下发幂等 | 复用既有 `_commit_allocation` 台账闭环，不新增写路径 |
| 安全 | 不触碰凭据 | 本 Story 纯映射引擎改动，无真机下发、无凭据读写（见 §10） |
| 性能 | 无退化 | `explicit_sw_reserved` 预计算 O(逻辑节点数)，透传跳过 O(1)/SW（见 §5.5） |

### 1.4 非目标（Out of Scope）

- 不改 Mock 语义（Mock 仍 `status='virtual'`、`physical=None`）。
- 不处理多 DUT（仍单 DUT）。
- 不改 `pool_merge.py`（SW 归并 + 台账映射为 STORY-038-02，technical-note）。
- 不改 exporter / switch_configurator / commands / topology_model / parser（后续 Wave）。

---

## 2. 文件影响范围

| 文件 | 所有权 | 操作 | 说明 |
|------|--------|------|------|
| `skills/topo-config/src/topo_mapper.py` | primary | 修改 | 核心改动（TASK-E1-01-01~04） |
| `skills/topo-config/tests/test_topo_mapper_sw.py` | 新建（本 Story） | 创建 | R-F-018 互斥 + POS/NEG/BND 单测（TASK-E1-01-05） |
| `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml` | 新建（本 Story） | 创建 | 显式 SW 场景测试造数 fixture（含 SW3=pppoe-server） |

**只读（不修改）**：`test_topo_mapper.py`、`test_topo_mapper_s4.py`（CR-037 node2 回归，由既有测试覆盖）、`physical_pool.py`、`physical_pool.yaml`（模板）。

**模块拆分与职责（本 Story 内）**：

| 对象 | 职责 | 变更 |
|------|------|------|
| `TopoMapper.match` | 预计算 `explicit_sw_reserved`，存入实例状态 | 新增 1 行 + 1 个 helper 调用 |
| `TopoMapper._compute_explicit_sw_reserved` | 纯函数：从 `candidates` 并集显式 SW 候选 | 新增 |
| `TopoMapper._try_l2_pass_through` | 透传候选剔除 `explicit_sw_reserved` | 新增 1 个 `continue` 跳过条件 |
| `TopoMapper._build_result` | 未映射节点读 `_build_unmapped_reason` 生成结构化 reason | 替换 unmapped reason 逻辑 |
| `TopoMapper._build_unmapped_reason` | 确定性分类：无设备 / 无候选 / 全占用 / 连通性失败 | 新增 |

---

## 3. 接口设计

### 3.1 内部接口（非公开 API，无跨模块契约变更）

| 接口 | 签名 | 变更 | 消费方 |
|------|------|------|--------|
| `TopoMapper.match(logical_topo, topo_limit=None, port_requirement=None, env_name=None)` | 不变 | 内部新增 `self.explicit_sw_reserved` / `self._candidates` 实例状态 | 外部调用方（exporter/编排）无感 |
| `_compute_explicit_sw_reserved(logical_topo, candidates) -> set[str]` | 新增 | 输入：逻辑 topo + `_build_candidates` 结果；输出：物理 SW 名集合 | 仅 `match` 内部 |
| `_try_l2_pass_through(dev_a, dev_b, link_id, used_ports, assignment, requirement=None, strict=True)` | 签名不变 | 内部读 `self.explicit_sw_reserved` 跳过 | `_check_connectivity` 内部 |
| `_build_unmapped_reason(nid, node, assignment) -> str` | 新增 | 输入：节点 id + 逻辑节点 + 当前 assignment；输出：结构化 reason 字符串 | `_build_result` 内部 |
| `DeviceMapping.role` | 不变（已存在） | 确认 `role=node.role` 传递 | exporter / switch_configurator（下游读 role 判定 PPPoE Server） |

**关键决策——`explicit_sw_reserved` 传递形态**：

- **主选**：实例属性 `self.explicit_sw_reserved`（`__init__` 初始化为 `set()`，`match()` 内回填）。
- **理由**：现有代码已用实例属性承载 match() 期间状态（`self._auto_inserted`、`self._degraded`、`self.logical_topo`）；避免穿透 `_check_connectivity → _try_l2_pass_through` 两层参数。
- **备选**：显式参数 `explicit_sw_reserved` 从 `match → _backtrack → _check_connectivity → _try_l2_pass_through` 穿透。切换条件：若未来要求 `_try_l2_pass_through` 可被外部独立复用且不依赖实例状态。

### 3.2 输入 / 输出

- 输入：`Topology` 对象（`NodeInfo.node_type='SW'`、`NodeInfo.role='pppoe-server'`、`NodeInfo.min_ports`）+ `PhysicalPool`（`node_type='SW'` 物理设备 + cables）。
- 输出：`MappingResult.device_mappings[]`，其中显式 SW 节点 `status='matched'`、`role=node.role`；自动透传 SW `status='auto_inserted'`、`role='l2-pass-through'`。

---

## 4. 数据模型

### 4.1 `explicit_sw_reserved` 集

- 类型：`set[str]`，元素为物理 SW 设备 `name`。
- 计算时机：`match()` 内、`_backtrack` 之前（回溯前预计算）。
- 计算规则（确定性，顺序无关）：
  ```text
  explicit_sw_reserved = ⋃ { candidates[nid] | 逻辑节点 nid 的 types 含 'SW' }
  其中 types = node.node_types or ([node.node_type] if node.node_type else [])
  candidates = _build_candidates(logical_topo)  # 已含 min_ports/device_type/约束/faulty/free 过滤
  ```
- 生命周期：每次 `match()` 调用重新计算，不跨调用持久化；`__init__` 初始化为空集。
- 保守保留语义：显式 SW 节点后续匹配失败时，其候选**不释放**回自动透传池（ADR-CR038-01 决策），这是互斥不变量成立的前提。

### 4.2 `DeviceMapping.role`

- 字段已存在（`role: Optional[str] = None`）。
- 显式 SW（及所有 matched 节点）：`role = node.role`（现状 `_build_result` 已实现，本 Story 验证 + 补回归单测，不改字段结构）。
- 自动透传 SW：`role = 'l2-pass-through'`（现状已实现，固定值，与显式 role 区分）。
- 台账写回：`_commit_allocation` 不涉及 role，role 仅在 `DeviceMapping` 结果对象传递，供下游 `switch_configurator` 判定 PPPoE Server（HLD §12.3）。

### 4.3 持久化

- 本 Story 无新增持久化 schema；台账占用沿用 `_commit_allocation` 写 `port_status[port].allocated_to=env_name`。

---

## 5. 核心流程（显式 SW 匹配 + 自动透传互斥）

### 5.1 主流程（`match()` 内新增两步）

```text
1. 解析/归一化（现状不变）：port_requirement、logic_links
2. candidates = _build_candidates(logical_topo)                 # 现状
3. self._candidates = candidates                                # 新增：供 _build_result 分类失败
4. self.explicit_sw_reserved = _compute_explicit_sw_reserved(...) # 新增：TASK-E1-01-01
5. ordered = _order_logical_nodes(...)                          # 现状（排序不变）
6. ok = _backtrack(ordered, candidates, assignment, used_ports, ...)  # 现状
7. result = _build_result(...)                                  # 现状（reason 增强）
8. env_name 且 matched/partial → _commit_allocation(...)         # 现状
```

### 5.2 显式 SW 匹配路径（现状已支持，本 Story 验证）

- `_build_candidates` 对 `node_type='SW'` 逻辑节点产出物理 SW 候选（`get_devices_by_type(['SW'])` + `min_ports` + 型号/约束过滤）。
- `_backtrack` 按既有逻辑匹配该 SW 节点；`_check_connectivity` 对 SW↔邻居校验直连 cable / 端口类型。
- 匹配成功后 `_build_result` 走 matched 分支，`role=node.role`。

### 5.3 自动透传互斥（核心新增，TASK-E1-01-02）

`_try_l2_pass_through` 的候选 SW 集合由现状的「全部 SW − assigned_sws」收紧为：

```text
候选 SW = get_devices_by_type('SW') − assigned_sws − explicit_sw_reserved − faulty_sws
```

实现落点：在 `for sw in sw_devices:` 循环内、`assigned_sws` 与 `is_faulty` 判定之后，插入：

```python
if sw_name in self.explicit_sw_reserved:
    continue   # 显式 SW 候选保守保留，不参与自动透传
```

该跳过条件**不依赖回溯进行到哪一步**，因为 `explicit_sw_reserved` 是回溯前预计算的全量集合。

### 5.4 互斥不变量（顺序无关性）

- **不变量**：`∀ link_id, auto_info = _auto_inserted[link_id] ⇒ auto_info['switch'] ∉ explicit_sw_reserved`。
- **顺序无关性论证**：
  1. `explicit_sw_reserved` 由 `_build_candidates` 结果集合并集得到，而 `_build_candidates` 只依赖 `logical_topo` + `pool`，与 `_order_logical_nodes` 排序无关；
  2. `_try_l2_pass_through` 跳过条件只读该预计算集合，与 `_backtrack` 当前 `idx` / `assignment` 无关；
  3. 因此无论显式 SW 节点在排序中位于 TG/DUT 之前或之后，自动透传都看不到其候选，互斥不变量恒定成立。
- 与现状 `assigned_sws` 的区别：`assigned_sws` 只能挡住「已被回溯赋值」的 SW（顺序相关），`explicit_sw_reserved` 挡住「未来可能被显式 SW 占用」的全部候选（顺序无关）。

### 5.5 技术设计细节

- **预计算开销**：`_compute_explicit_sw_reserved` 遍历逻辑节点一次、每个节点做 `'SW' in types` 常量判断 + 集合并集，O(节点数 × 候选数)，量级微小。
- **透传跳过开销**：每次 SW 迭代多一次 `in set` 判断，O(1)。
- **回滚安全**：`explicit_sw_reserved` 在 `_backtrack` 回滚路径中不修改（预计算只读），回滚无需处理它，与 `_auto_inserted` 回滚逻辑正交。
- **faulty 一致性**：`_build_candidates` 已剔除 `is_faulty` 的 SW，故 `explicit_sw_reserved` 天然不含 faulty SW；透传侧同样 `is_faulty` 跳过，两侧一致。

---

## 6. 异常处理（结构化 reason，R-NF-002）

### 6.1 失败分类（`_build_unmapped_reason`，确定性优先级）

`_build_result` 未映射分支（`phys is None` 且非 Mock）改为调用 `_build_unmapped_reason(nid, node, assignment)`，按以下**从上到下第一个命中**分类：

| 优先级 | 条件 | 返回 reason | 对应场景 |
|--------|------|-------------|---------|
| 1 | `types` 为空 | `无 node_type 声明，无法匹配` | 解析缺陷 |
| 2 | `pool.get_devices_by_type(types)` 为空 | `无 {types} 类型物理设备（pool 未配置）` | SCN-038-NEG-001（pool 无 SW） |
| 3 | `self._candidates.get(nid)` 为空 | `无满足条件的空闲物理设备（端口不足或设备约束/型号不满足）` | SCN-038-NEG-001 变体（有 SW 但端口不足） |
| 4 | 候选全部 ∈ `assignment.values()` | `所有候选物理设备已被其他逻辑节点占用` | 资源竞争 |
| 5 | 其他（候选非空但连通性校验失败） | `无满足连通性约束的空闲物理设备（接口/端口类型不匹配或无空闲直连 cable）` | SCN-038-NEG-002（接口不匹配） |

- 分类 2/3 覆盖「SW 无候选」，分类 5 覆盖「接口不匹配」，二者 `reason` 字符串可被测试断言区分。
- degradable 节点保持现状专属 reason（降级建议），不套用上述通用分类。
- 该分类只依赖 `self._candidates`（回溯前算定）与 `assignment`（回溯结果），确定性成立。

### 6.2 自动透传无候选

- `_try_l2_pass_through` 剔除 reserved 后无可用 SW → 返回 `None` → `_check_connectivity` 返回 `None` → 触发回溯换候选 / 最终归入分类 5 reason。显式 SW 与自动透传**不冲突**（reserved 已剔除，不触发「冲突」失败路径）。

---

## 7. 测试设计

### 7.1 测试入口

- 命令：`uv run --python 3.11 pytest skills/topo-config/tests/test_topo_mapper_sw.py -v`
- 全量回归：`uv run --python 3.11 pytest skills/topo-config/tests/`

### 7.2 新 fixture：`skills/topo-config/tests/fixtures/sw_explicit.sample.yaml`

- 设备：`TG1`(node_type=TG)、`dut1`(node_type=DUT)、`SW1`/`SW2`/`SW3`(node_type=SW，SW3 带 `role: pppoe-server`)。
- 线缆构造两态：
  - **显式 SW 场景**：TG1↔SW1、SW1↔dut1、dut1↔SW2、SW2↔SW3、SW3↔TG1（环回，全部 SW 显式）。
  - **透传互斥场景**：TG1 与 dut1 无直连 cable，仅 TG1↔SW1、SW1↔dut1、TG1↔SW2、SW2↔dut1（两条透传候选路径），再挂显式 SW 节点抢占其一。
- 台账 fixture 复用既有 `ledger_*.sample.yaml` 的 `fresh_ledger` 清空模式。

### 7.3 用例清单（TASK-E1-01-05）

| 用例 | 覆盖场景 | 断言（量化） |
|------|---------|-------------|
| `test_explicit_sw_matched_and_role` | SCN-038-POS-001/002 | 显式 SW 节点 `status='matched'`；`DeviceMapping.role=='pppoe-server'`；`link_mappings` 端点含该 SW |
| `test_sw_no_candidate_reason` | SCN-038-NEG-001 | pool 无 SW → SW 节点 `status='unmapped'`，`reason` 含 `无 SW 类型物理设备` |
| `test_sw_interface_mismatch_reason` | SCN-038-NEG-002 | SW 候选非空但无匹配 cable/端口 → `status='unmapped'`，`reason` 含 `连通性约束` |
| `test_mock_still_virtual` | SCN-038-BND-002 | Mock `status='virtual'`、`physical=None`；含 Mock 端点 link `status='unmapped'`、`reason` 含 `Mock` |
| `test_explicit_sw_reserved_excludes_passthrough` | R-F-018 核心 | 设置 `mapper.explicit_sw_reserved={'SW2'}` 后直接调 `_try_l2_pass_through`，返回 `info['switch'] != 'SW2'`；端到端 `match()` 中每台自动透传 SW 均 `∉ explicit_sw_reserved` |
| `test_explicit_sw_reserved_order_invariance` | R-F-018 顺序无关 | 交换 `logical_topo.nodes` 插入顺序（或手工置换 `ordered`）跑两次 `match()`，两次 `explicit_sw_reserved` 相等，且 `auto_inserted[*].switch ∩ explicit_sw_reserved == ∅` 恒成立 |
| `test_cr037_node2_regression` | R-NF-004 | node2 场景（无显式 SW）`status='matched'`，`explicit_sw_reserved` 为空，自动透传行为与现状一致 |

### 7.4 通过标准

- 上述 7 用例全 PASS。
- `skills/topo-config/tests/` 既有测试（含 `test_topo_mapper_s4.py`）全 PASS，无回归。

---

## 8. 实施步骤（最小切片）

按 TASK-ID 顺序，每个切片可独立提交 + 可独立验证：

| 切片 | TASK-ID | 内容 | 验证方式 |
|------|---------|------|---------|
| S1 | TASK-E1-01-01 | `__init__` 增 `self.explicit_sw_reserved=set()`、`self._candidates=None`；新增 `_compute_explicit_sw_reserved`；`match()` 在 `_build_candidates` 后回填两状态 | 单测断言 reserved 集正确 |
| S2 | TASK-E1-01-02 | `_try_l2_pass_through` 循环内新增 `if sw_name in self.explicit_sw_reserved: continue` | `test_explicit_sw_reserved_excludes_passthrough` |
| S3 | TASK-E1-01-03 | 确认 `_build_result` matched 分支 `role=node.role` 已生效；无需改字段（若审查发现缺口则补） | `test_explicit_sw_matched_and_role` |
| S4 | TASK-E1-01-04 | 新增 `_build_unmapped_reason`；`_build_result` unmapped 分支改调该方法 | `test_sw_no_candidate_reason` / `test_sw_interface_mismatch_reason` |
| S5 | TASK-E1-01-05 | 新建 fixture + 单测文件，补齐 7 用例 | 全量 pytest PASS |

> 切片间无跨 Story 依赖；S1→S2 有顺序（S2 读 S1 的状态），S3/S4 可与 S1/S2 并行，S5 收尾。

---

## 9. 回滚策略

- **代码回滚**：本 Story 改动全部集中在 `topo_mapper.py` + 新增测试/fixture 文件，可单文件 `git revert` 或整体撤销，不影响其他 Story。
- **行为回滚**：删除 `explicit_sw_reserved` 后，`_try_l2_pass_through` 退化为现状（仅跳过 `assigned_sws`），即 ADR-CR038-01 记录的备选方案「仅跳过已分配 SW」。
- **数据回滚**：台账占用沿用既有 `release_env(env_name)` 释放闭环（本 Story 不新增持久化 schema），无需额外数据迁移。
- **发布回滚**：若 R-F-018 互斥在真机环回场景出现「SW 候选过度保守」导致匹配失败，切回备选（仅跳过已分配 SW），显式 SW 与透传改为依赖回溯顺序——该切换条件写入 ADR-CR038-01「回退/切换」。

---

## 10. 安全约束

- **凭据**：本 Story 不读取 / 不写入任何凭据；`password` 仅在 fixture 中用 `test-placeholder` 占位。
- **真机下发**：本 Story 纯映射引擎，无 telnet / web / trex 调用，无 runtime_authorization 需求。
- **台账写入**：`_commit_allocation` 仅在 `env_name` 非空且 `matched/partial` 时写回，沿用现状幂等语义（`allocate` 失败跳过 + warning）。
- **GE1_1~4 禁改动**：本 Story 不改 `physical_pool.yaml` 模板与任何 GE 实例端口，无触碰。
- **无新增外部接口**：不改变 `match()` 对外签名与返回 schema，无攻击面扩大。

---

## 11. 契约映射（需求 → 实现点）

| 需求 | 实现点（文件:函数:切片） | 验证入口（测试） |
|------|--------------------------|------------------|
| R-F-001 SW 显式匹配 | `topo_mapper.py:_build_candidates/_backtrack`（现状）+ S3 确认 | `test_explicit_sw_matched_and_role` |
| R-F-002 role 保留 | `topo_mapper.py:_build_result`（matched 分支 `role=node.role`）+ S3 | `test_explicit_sw_matched_and_role` |
| R-F-018 显式优先 | `topo_mapper.py:_compute_explicit_sw_reserved`(S1) + `_try_l2_pass_through` 跳过(S2) | `test_explicit_sw_reserved_excludes_passthrough` + `test_explicit_sw_reserved_order_invariance` |
| R-NF-002 结构化 reason | `topo_mapper.py:_build_unmapped_reason`(S4) | `test_sw_no_candidate_reason` + `test_sw_interface_mismatch_reason` |
| R-NF-004 回归无破坏 | reserved 默认空集（无显式 SW 拓扑零影响） | `test_cr037_node2_regression` + 既有 `test_topo_mapper*.py` 全 PASS |

**TASK-ID ↔ 文件影响一一对应**：TASK-E1-01-01~04 → `topo_mapper.py`；TASK-E1-01-05 → `tests/test_topo_mapper_sw.py` + `tests/fixtures/sw_explicit.sample.yaml`。

---

## 12. 依赖与前置

| 依赖 | 类型 | 状态 | 说明 |
|------|------|------|------|
| HLD-CR-038 §10.1 | 设计基线 | 已存在（status=draft，CP3 已批准） | 显式 SW 优先级算法来源 |
| ADR-CR038-01 | 架构决策 | 已存在 | reserved 集互斥决策 + 回退方案 |
| `docs/features/cr038-sw-mapping/DESIGN.md` | Feature 设计 | 已存在 | §2 确定性算法 + §3 role + §4 台账互斥 |
| STORY-038-01 前置 Story | `depends_on: []` | 无 | W1 起点 |
| 文件所有权 | `primary: topo_mapper.py` | 无冲突 | 本 Story 独占 primary 文件，测试文件为新建 |

**前置条件（进入实现前必须满足）**：
1. `design_evidence_confirmed=true`（CP5 全量人工确认通过，含本 LLD）。
2. Story `status=dev-ready`（或 `lld-approved`）。
3. `dev_gate=cp5-approved` 满足。
4. `topo_mapper.py` 未被其他 `dev_running` Story 占用（文件所有权无冲突）。

---

## 13. 风险与开放项

### 13.1 风险

| 风险 | 影响 | 应对 |
|------|------|------|
| reserved 过度保守 | 显式 SW 匹配失败时其候选不释放，可能导致「SW 不足→整体失败」 | 明确为 ADR-CR038-01 决策；如需复用改回「仅跳过已分配」，切换条件已记录 |
| 顺序无关性被破坏 | 若后续有人把 reserved 计算移入回溯循环 | 单测 `test_explicit_sw_reserved_order_invariance` 守护 |
| reason 分类误判 | 资源竞争（分类 4）与连通性失败（分类 5）边界 | 用 `assignment` 精确判定「候选是否已全被占用」，确定性分类 |

### 13.2 实现灰区与取舍记录

| 灰区 | 决策 | 理由 | 证据 | 重访条件 |
|------|------|------|------|---------|
| 透传方法命名 | 实际唯一入口为 `_try_l2_pass_through`（无 `_find_l2_passthrough`/`_find_l2_switch_pass_through`） | 代码 grep 确认 | `topo_mapper.py:546` | 无（历史别名已废弃） |
| `explicit_sw_reserved` 传递形态 | 实例属性（主选）/ 参数穿透（备选） | 与现有 match() 状态模式一致 | §3.1 | 需外部独立复用透传时切参数 |
| 测试文件所有权未在 Story 卡片 `file_ownership` 显式列出 | 新建文件规避冲突，不修改既有测试文件 | 新建 `test_topo_mapper_sw.py` 不与 CR-037/其他 Story 冲突 | Story 卡片 file_ownership | CP5 确认时核对测试文件归属 |

### 13.3 开放项

- 无 `blocks_lld=true` 的 clarification item（本 Story 预期无阻塞灰区，未写 QUESTION-LEDGER）。
- 无 OPEN / Spike。相关决策均已由 ADR-CR038-01 / Feature DESIGN 收敛为确定性契约。

---

## 14. 验收与交接

### 14.1 Definition of Done

- [ ] `topo_mapper.py` 完成 S1~S4 改动，`explicit_sw_reserved` 互斥生效。
- [ ] `test_topo_mapper_sw.py` + `sw_explicit.sample.yaml` 落盘，7 用例全 PASS。
- [ ] `uv run --python 3.11 pytest skills/topo-config/tests/` 全量 PASS（含 CR-037 node2 回归）。
- [ ] 5 条验收场景通过：SCN-038-POS-001/002、NEG-001/002、BND-002。
- [ ] R-F-018 互斥单测通过（顺序无关）。
- [ ] CP6 编码完成检查结果写入 `process/checks/CP6-STORY-038-01-sw-explicit-mapping-CODING-DONE.md`。

### 14.2 交接摘要（给 meta-qa）

- **实现文件**：`skills/topo-config/src/topo_mapper.py`（修改）+ `skills/topo-config/tests/test_topo_mapper_sw.py`（新建）+ `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml`（新建）。
- **验证入口**：`uv run --python 3.11 pytest skills/topo-config/tests/`。
- **重点风险提示**：`explicit_sw_reserved` 保守保留语义（显式 SW 失败不释放候选）；reason 四级分类边界（分类 4 vs 5）。
- **下游消费契约**：`DeviceMapping.role`（`pppoe-server` / `l2-pass-through`）供 STORY-038-03/04（switch_configurator）判定 PPPoE Server；`MappingResult` schema 不变。

---

> 本 LLD 仅产出设计证据，未修改 `topo_mapper.py` 或其他工程资产。代码改动待 CP5 全量确认通过、Story 进入 `dev-ready` 后，由实现阶段（story-execution）执行。
