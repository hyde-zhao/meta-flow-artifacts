---
doc_type: HLD
id: CR-037-HLD
cr_id: CR-037
version: "0.4"
status: draft
created_at: "2026-08-05"
owner: host-orchestrator（solution-design）
---

# HLD — ptm-te 集成 topo 管理功能（CR-037）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v0.1 | 2026-08-05 | host-orchestrator | 初稿：三层架构 + topo-planning 6 模块 + per-link 映射增强 + 产物导出 + 安装集成 |
| v0.2 | 2026-08-05 | host-orchestrator（评审修订） | §4.4 增加 ledger 占用闭环（H1）+ 约束叠加优先级（H4）；§4.3 增加 TG 归并（H2）；§4.2 增加多节点作用域 all 语义（H3）+ 空 limit 默认（O-a）；§4.5 增加 --report-out（O-c）；§10 明确 12 Story 计数（O-b）；§11 完成准则拆两层验证（H5） |
| v0.3 | 2026-08-05 | meta-dev（设计审查修订） | 事实修正：§1.2/§1.3/§11 标准拓扑数 13→12（P1-1，collection.md 实际 12 个 node* 拓扑）；§1.2/§4.2 硬件系列数 14→13（P1-2）；§4.1 find_topology_file 返回类型明确为 YAML 文件路径（P0-1）+ index 校验改为 collection 级条目（P0-2） |
| v0.4 | 2026-08-05 | host-orchestrator（CP5 第二轮复审） | §1.3 补充 MVP 边界声明（R1 范围决策）：本 CR 映射 MVP = 单 TG + 单 DUT 直连（node2_dut1_tg1_linkN 系列），多节点/含 SW 拓扑在 exporter 显式报 TOPO_NODE_COUNT_UNSUPPORTED（范围外，后续 CR 扩展）；不改架构 |

## 1. 背景与目标

### 1.1 背景

ptm-te（测试执行工程师）当前执行用例前需人工维护 `topology/topology.yaml`（port_mapping/nodes/links）。需求：集成 topo 管理能力，根据**用例逻辑 topo + 用户 topo-limit** 自动从物理资源池抽取空闲可用设备，完成逻辑 topo → 物理 topo 映射，产出 ptm-te 可直接消费的 env-file，可选真机下发。

核心场景："在海光3250上执行这个用例，其中需要包含 TE|10GE 链路"。

### 1.2 目标（量化成功标准）

1. 从用例「组网约束/组网描述」提取 `topology_id`，命中 `resource/network-topology/topology-collection.md` 的 12 个标准拓扑，**解析成功率 100%**（双源兼容）。
2. topo-limit（NLP → 结构化）支持设备类型约束 + 链路约束 3 模式（include-at-least-one / all / specific），平台别名经 manaul device-reference.md 完整映射（13 硬件系列）展开。
3. 映射结果与逻辑 topo **链路数一致**（不一致报错）；映射成功产出 env-file 满足 `${ENV.*}` 9 类占位符硬契约，case-execution 零适配消费。
4. 产物**不含任何凭据**（ADR-02）；测试造数隔离在 ptm-team 测试目录，**不注入 ptm-te 目标项目**。
5. 映射失败输出**结构化限制校验报告**（缺设备/缺端口/链路约束不满足），不兜底。

### 1.3 非目标（Out of Scope）

- **不做真实设备登记**：hg3250 等设备登记由 ptm-te 目标项目用户维护，ptm-team 只提供模板与 schema。
- **不做逻辑 topo 集合扩展**：12 个标准拓扑来自 `topology-collection.md`（共享资源），本 CR 只消费。
- **不改 ptm-tde 上游**：组网约束字段由 ptm-tde 产出，本 CR 只消费其 topo_id 格式。
- **不做全自动 NLP**：NLP→limit 转换由 agent 实现，CLI 只收结构化参数（AG-4）。
- **MVP 边界（R1，v0.4）**：本 CR 映射 MVP = **单 TG + 单 DUT 直连**（`node2_dut1_tg1_linkN` 系列）；多节点（≥2 DUT）或含 SW/Mock/PC 拓扑在 exporter 显式报 `TOPO_NODE_COUNT_UNSUPPORTED`（范围外，后续 CR 扩展）。

## 2. 现状与约束

- **现有系统**：topo-config（TopoMapper/EnvironmentManager/EnvironmentDeployer，`physical_pool.yaml` 当前为示例数据）；case-execution（env-file 消费，`${ENV.*}` 9 类占位符）；device-management（devices.yaml + device-reference.md 型号表）；install.py（PTM_TE_SKILLS 5 个 + resource 安装）。
- **三层文件结构**（用户认可）：层1 模板（ptm-team）/ 层2 共享资源（`~/.ptm-team/resource/network-topology/`）/ 层3 目标项目真实数据（`topology/physical_pool.yaml` + `devices.yaml` + env-file，用户维护）。
- **评审核实**：E1（流程 [1]-[8] 无 [0]）、E2（组网约束优先/组网描述回退）、E3（collection.md 路径）、E4（9 类占位符）、G1-G6、O1-O3 已写入 CR-037 §事实核对结论。

## 3. 总体架构

```
┌─ ptm-team（设计+模板+引擎+skill）─────────────────────────────┐
│  topo-planning skill（新建）                                   │
│   ├─ ① topology_collection.py  —— collection.md YAML 提取+索引│
│   ├─ ② limit_parser.py          —— NLP→结构化 limit+平台展开  │
│   ├─ ③ pool_merge.py            —— devices×pool 归并          │
│   ├─ ④ exporter.py              —— 映射→env-file+命名+报告    │
│   └─ CLI topo-plan（结构化入参，agent 负责 NLP→CLI）           │
│  topo-config 引擎（增强）                                      │
│   ├─ topo_mapper.py  —— per-link port_requirement + hardware  │
│   ├─ config/physical_pool.yaml（模板化）+ physical_ledger.yaml │
│  install.py —— PTM_TE_SKILLS +首次生成模板副本(不含造数)       │
└────────────────────────────────────────────────────────────────┘
             │ 安装
┌─ ptm-te 目标项目（真实数据，用户维护）─────────────────────────┐
│  topology/physical_pool.yaml + physical_ledger.yaml + devices │
│  cases/ + runs/ + input/                                     │
└────────────────────────────────────────────────────────────────┘
```

**数据流**：
```
用例 PC → 组网约束/组网描述 → ① topo_id → collection.md → 逻辑 Topology
用户 NLP → agent → limit 结构化 → ② platform 展开 ──────────┐
topology/physical_pool.yaml + devices.yaml → ③ 归并池 ──────► TopoMapper.match
                                                            │
                    per-link 约束（回溯剪枝）←── ② 链路约束   │
                                                            ▼
                                          ④ exporter → env-file（${ENV.*} 9 类）
                                          ④ 结构化失败报告（映射失败时）
                                          （可选）EnvironmentDeployer 真机下发
```

## 4. 关键模块设计

### 4.1 topology_collection.py（新建，S1）

- 解析 `resource/network-topology/topology-collection.md`：提取 `## {topo_id}` 后第一个 YAML 代码块。
- 提供 `find_topology_file(topo_id)` → 返回 **YAML 文件路径**（写入标准缓存路径，与 `parser.py` 的 `detect_topology_format(filepath)` + `_parse_yaml_topology(filepath)` 消费契约一致，P0-1）；读 `index.yaml` 校验 collection 级资源条目 `tgfw-topo-collection` 的 `compatible_consumers` 含 ptm-te（不要求 node* 顶层条目，P0-2）。
- 补齐 topo-config `parser.parse_topology(topo_id=...)` 的缺失依赖（`from topology_collection import find_topology_file`）。
- 逻辑 topo 解析后**从 interfaces 推导 min_ports**（collection 无 min_ports 字段，接口数即端口需求）。

### 4.2 limit_parser.py（新建，S2）

- NLP → 结构化 topo-limit（agent 转换，交付 limit 语法规范文档含示例/反例集）：
  ```yaml
  topo_limit:
    device_constraints: [{node_ref, node_type, platform, device_type}]
    link_constraints:
      - {mode: include-at-least-one|all|specific, link, speed_class, media_type}
  ```
- **平台别名展开**：消费 manaul `device-reference.md` 完整映射表（13 硬件系列 → 型号 + 默认匹配规则 + 型号冲突消歧；冲突型号如 A1500-HU 结合 devices.yaml 实际设备 hardware_platform 等价类消歧）。
- **多节点作用域语义（H3）**：`device_constraints.node_ref` 指定 → 仅作用于该逻辑节点；`node_ref` 空 → 作用于**全部**该 `node_type` 节点（all 语义，如逻辑 topo 多 DUT 且 limit="海光3250" 时所有 DUT 均须为海光3250 型号）。语法规范显式定义 all/any 语义与示例/反例，保证可测试。
- **空 limit 默认行为（O-a）**：无 topo-limit（不传 --limit/--limit-file）时跳过约束匹配，等价现有 topo-config 行为（仅按 node_type + min_ports 匹配），向后兼容。
- **端口兼容矩阵校验**：GE=copper\|fiber；TE/XTE/QTE=fiber-only；非法组合报 PARAM_INVALID。

### 4.3 pool_merge.py（新建，S3）

- 加载目标项目 `topology/physical_pool.yaml` + `topology/devices.yaml`。
- 归并规则（AG-3 推荐 A）：**host 关联** `device_groups[].firewall.host ↔ pool.nodes[].management.host`；池承载接线/端口/台账，devices 覆盖设备型号/平台/管理地址；冲突（同 host 型号不同）以 devices.yaml 为准并 warning。
- **TG 归并（H2）**：`device_groups[].tg.api_server`（或 tg.host）↔ `pool.nodes[].management.api_server` 关联池内 TG 节点；池 TG 节点补充 `sub_type`（ixia-c/trex）与 `hardware_platform` 供映射；TG 无 firewall 块，归并规则单独定义。
- 台账三角映射：`台账.model ↔ 池.device_type ↔ device-reference 平台`。
- 池全量承载时降级为"仅校验 devices 存在性"（AG-3 备选 B）。

### 4.4 topo_mapper.py 增强（S4）

- `match(topo, port_requirement=None)` → 增加 per-link 约束：`port_requirement: {link_id: {speed_class, media_type}}`。
- 回溯匹配阶段在 `_check_connectivity` / `_port_meets_requirement` 按 link_id 取对应约束（AG-2 推荐 A，提前剪枝）。
- **约束叠加优先级（H4）**：用户 topo-limit 约束 > 逻辑 topo（collection）自带链路约束 > 默认（无约束）。冲突时用户约束优先，逻辑 topo 自带约束为下限（如 collection link 声明 GE，用户可加 TE 上限）；语法规范与 S4 单测覆盖叠加顺序。
- **空闲端口跨映射占用闭环（H1，P0）**：`_is_port_busy` 消费 ledger `allocated_to`（跨用例持久占用，非仅本次 `used_ports`）；**映射成功后写回 ledger 占用**（`allocate`：allocated_to=env_name + allocated_vlan），环境删除/teardown 时释放。避免两个用例映射到同一物理设备产生冲突。此闭环为 S4 full-lld 必需设计项。
- 物理池节点增加 `hardware_platform` 字段匹配（与 device_type 并列；device-reference 平台展开）。
- 端口兼容矩阵：GE 口满足 copper/fiber 请求，TE/XTE/QTE 仅 fiber；无 TE 口时 TE 约束判 failed。

### 4.5 exporter.py（新建，S5）

- 映射结果（设备/端口/link + IP 规划）→ env-file：`metadata + port_mapping + nodes + links`，满足 case-execution `${ENV.*}` 9 类占位符硬契约（见 §7）。
- **命名**：默认 `topology/topology.yaml`；`--output` 自定义描述性命名（如 `node2_dut1(nxp290-171_10.113.55.171)_tg1(trex_10.113.55.175)_link3.yml`）。
- **link 数一致性校验**：实际链路数必须 = 逻辑 topo 链路数，不一致报错。
- **凭据禁止**：产物不含密码（ADR-02），管理地址仅 host；凭据走 ptm-atomic auth / devices.yaml `${ENV_VAR}`。
- **结构化失败报告**：JSON + 人读，列出缺设备/缺 TE 端口/链路约束不满足明细（AG-6）。CLI 支持 `--report-out <path>` 输出报告文件（缺省 stdout），报告可作为 CP7 验证证据（O-c）。

### 4.6 IP 规划（S6）

- 复用 topo-config IPAM（198.18.0.0/15 自动分配 or per-broadcast-domain 指定），广播域按导出器计算（含 DUT-DUT 直连 L3 域）。

### 4.7 topo-planning skill + CLI（S7）

- skill 向导 5 步（展示池 → 选/建逻辑 topo → 映射 → 导出 → 部署/验证）。
- CLI：`topo-plan --topo-id/--case-file --limit/--limit-file --devices-yaml --pool-path --ledger-path --output --dry-run`；默认 dry-run。
- NLP→CLI 转换由 agent 实现（G5）；limit 语法规范为显式交付物。

### 4.8 ptm-te 编排 + install.py 集成（S8）

- ptm-te 编排流程 **[1] 用例解析 → [2] 设备准备之间**插入拓扑映射步骤（E1）；topo_id 提取优先「组网约束」、回退「组网描述」括号（E2）。
- `PTM_TE_SKILLS` 增加 `topo-config` + `topo-planning`（方案 A）。
- install.py 首次安装生成目标项目 `topology/physical_pool.yaml` + `physical_ledger.yaml` **模板副本（不含造数）**（AG-7 推荐 A，幂等跳过已有文件）。

### 4.9 可选真机部署（S9）

- 复用 `EnvironmentDeployer`；`--execute` 为独立 `runtime_authorization` 决策项（DQ-037-04，O3），不在 CP 批准隐含。

## 5. 关键架构决策（ADR 摘要）

| ADR | 决策 | 引用 |
|-----|------|------|
| ADR-CR037-01 | topo-planning 独立 skill + import topo-config 引擎（方案 A） | AG-1 / DQ-037 用户确认 |
| ADR-CR037-02 | per-link 端口约束回溯剪枝接入 | AG-2 |
| ADR-CR037-03 | 三层文件结构；真实数据用户维护；测试造数隔离 | 用户认可/纠正 |
| ADR-CR037-04 | 平台别名消费 manaul device-reference.md 完整映射 | DQ-037-02 |
| ADR-CR037-05 | 产物=env-file（默认 topology.yaml + 可选描述性命名），link 数一致性 | O1 / 用户确认 |
| ADR-CR037-06 | 映射失败结构化报告，不兜底 | O2 / 用户指示 |
| ADR-CR037-07 | install.py 首次生成模板副本（不含造数） | AG-7 |

## 6. 架构灰区决策（AG-1~7）

AG-1/2/5/6 已闭合（§4 采用推荐）；AG-3/4/7 为开放项，推荐方案已列（§4.3/4.2/4.8），列入 CP3 待确认决策。

## 7. 产物契约（${ENV.*} 9 类，ADR-09 硬契约）

| 占位符 | 解析路径 |
|---|---|
| `${ENV.tg.port1}` / `${ENV.dut.port1}` | port_mapping.port1.tg / .dut |
| `${ENV.tg.port1.ip}` / `.gw` | nodes.tg1.interfaces.port1.ip / .gw |
| `${ENV.dut.port1.ip}` | nodes.dut1.interfaces.port1.ip |
| `${ENV.tg.url}` | nodes.tg1.trex_api_url |
| `${ENV.dut.url}` | nodes.dut1.host |
| `${ENV.dut.next_hop}` | nodes.dut1.next_hop |
| `${ENV.tg.ports[port1,port2]}` | [port_mapping.port1.tg, port_mapping.port2.tg] |

端口名必须加引号（YAML 1.1 整数陷阱，Gotcha #11）。

## 8. NFR 与安全

- **凭据**：产物/env-file 禁止密码（ADR-02）；物理池模板用 `${PLACEHOLDER}`。
- **隔离**：测试造数隔离在 ptm-team `tests/fixtures`，安装产物不含造数；目标项目真实数据 `.gitignore` 忽略。
- **幂等**：install 生成模板副本幂等（已有则跳过）。
- **确定性**：映射失败输出结构化报告；链路数一致性强制校验。

## 9. 风险与回退

| 风险 | 影响 | 回退 |
|------|------|------|
| 物理池无 TE 口 → TE 约束失败 | 核心场景（海光3250+TE）映射失败 | 预期行为，结构化反馈；目标项目补 TE 口后重试 |
| AG-3 归并冲突 | 型号不一致 | 以 devices.yaml 为准 + warning；池全量时降级 B |
| AG-7 install 生成覆盖 | 覆盖用户已填数据 | 幂等：已有文件跳过 |
| per-link 剪枝复杂度 | 回溯性能 | 全局约束退化为现有路径 |

## 10. Story 拆解（输入到 DEVELOPMENT-PLAN.yaml）

> **Story 计数（O-b）**：12 个执行 Story = P-1 + S1-S11；P-2/P-3 不单列执行 Story，**并入 S8（模板副本生成）与 S3（台账三角映射）**，避免与 DEVELOPMENT-PLAN 计数歧义。

| Wave | Story | 模块 | lld_policy |
|------|-------|------|-----------|
| 前置 | P-1 物理池模板化 + 造数隔离 | config/physical_pool.yaml + tests/fixtures | technical-note |
| 前置 | P-2 目标项目模板副本生成机制 | install.py（并入 S8） | waived（并入 S8） |
| 前置 | P-3 台账三角映射 | pool_merge.py（并入 S3） | waived（并入 S3） |
| 1 | S1 topology_collection.py | 新建 | full-lld |
| 1 | S2 limit_parser.py（含语法规范） | 新建 | full-lld |
| 1 | S3 pool_merge.py | 新建 | full-lld |
| 2 | S4 topo_mapper per-link + hardware_platform | 增强 | full-lld |
| 2 | S5 exporter.py（env-file+命名+报告） | 新建 | full-lld |
| 2 | S6 IP 规划集成 | 复用 | technical-note |
| 3 | S7 topo-planning skill + CLI | 新建 | full-lld |
| 3 | S8 ptm-te 集成 + install.py | 修改 | full-lld |
| 3 | S9 可选部署（--execute 授权） | 复用 | technical-note |
| 4 | S10 测试（单测+参考用例 dry-run） | tests/ | technical-note |
| 4 | S11 文档（SKILL/执行指导/limit 语法） | docs | waived |

## 11. 完成准则（量化，分两层验证 — H5）

### 层 A：fixture 验证（必达，ptm-team 测试造数）

> 数据来源：ptm-team `tests/fixtures/` 造数（含 hg3250 假设设备 + TE 端口），隔离于目标项目。

1. 12 个标准拓扑 `find_topology_file` 命中率 100%（返回 YAML 文件路径）；`node2_dut1_tg1_link3` 双源提取（组网约束/组网描述）均通过。
2. limit 解析：设备约束（含多节点 all 语义）+ 3 种链路约束模式 + 平台别名展开（≥5 别名含海光3250）+ 空 limit 默认行为单测通过。
3. per-link TE 约束：fixture 池含 TE 口时映射成功；不含时返回结构化失败报告（缺 TE 端口）+ `--report-out` 落盘。
4. **ledger 占用闭环（H1）**：映射成功后 `allocated_to` 写回；二次映射避开已占用端口；释放后恢复 free（单测覆盖）。
5. 约束叠加优先级（H4）：用户约束 > 逻辑 topo 自带 > 默认，单测覆盖。
6. 产物 env-file 通过 case-execution `load_env_file` 校验；链路数一致性错误路径有单测。
7. 安装 dry-run：`install --agent ptm-te` 安装 topo-config + topo-planning + 生成模板副本（不含造数）。
8. 参考用例 `IPv4策略路由`（TOPO-01，fixture 池）映射 → env-file → dry-run 执行链路打通。

### 层 B：目标项目真实数据验证（可选，依赖 P-2）

> 前置：ptm-te 目标项目 `topology/physical_pool.yaml` + `physical_ledger.yaml` 已由用户填写真实数据（含 hg3250 等）。P-2 未就绪时本层标注未验证风险，不阻断 CP7 fixture 验证结论。

9. 目标项目真实池映射：`IPv4策略路由` 用例组网约束 → 真实设备 → env-file。
10. （可选）真实 dry-run/`--execute` 执行验证（--execute 需 DQ-037-04 独立授权）。
