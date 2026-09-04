---
story_id: "CR037-S6"
story_slug: "ip-planning"
cr_id: "CR-037"
title: "IP 规划集成（复用 topo-config IPAM，per-broadcast-domain）"
priority: "P1"
wave: 2
status: "lld-ready"
depends_on:
  - "S5"
dependency_type: "precedence"
assignee: null
dev_context: "在 exporter.py 内集成 topo-config 既有 IPAM（198.18.0.0/15 自动分配 or per-broadcast-domain 指定），广播域由 topology_exporter 计算（含 DUT-DUT 直连 L3 域），产出 ip_plan（权威 schema=ipam.ip_plan_to_dict）供 S5 exporter 渲染 env-file 的 IP/gw/next_hop 字段。不新建 IPAM 引擎，只做接入桥接。SW（L2）拓扑广播域 IP 规划依赖 vlan_assign（VlanIsolator 产出）；vlan_assign 为 None 且拓扑含 SW 时**显式报错**（IP_PLAN_SW_VLAN_MISSING），不得静默跳过 L2 域产出假成功 ip_plan。"
validation_context: "IP 规划覆盖：自动模式（198.18.0.0/15 按序分配 /24）+ preset 模式（per-broadcast-domain 指定子网）；广播域计算复用 topology_exporter（含 L3 直连域）；容量不足（端点超子网容量）报结构化错误。产物 env-file 的 IP 字段可被 case-execution resolve_env_refs 解析。"
acceptance_criteria:
  - "compute_ip_plan(mapping_result, logical_topo, pool, vlan_assign=None, preset_subnets=None) 输出 ip_plan dict（权威 schema=ipam.ip_plan_to_dict，对齐 ipam.IPAssignment）"
  - "自动模式：从 198.18.0.0/15 按序分配 /24 子网，无交互"
  - "preset 模式：per-broadcast-domain 指定子网，重复/非法子网报错"
  - "广播域计算复用 topology_exporter._compute_broadcast_domains（含 DUT-DUT 直连 L3 域）"
  - "容量不足（端点+网关 > 可用 IP）报结构化错误，不产出非法子网"
  - "SW（L2）拓扑且 vlan_assign=None：显式报错 IP_PLAN_SW_VLAN_MISSING，不产出缺 L2 域的假成功 ip_plan（不允许静默跳过 SW L2 域）"
  - "含 SW 拓扑校验：即使传 vlan_assign，若广播域结果缺 switch_ports 非空的 L2 域，报 IP_PLAN_SW_L2_DOMAIN_MISSING"
  - "ip_plan 渲染后 env-file 的 nodes.*.interfaces.*.ip/gw 与 dut.next_hop 可被 case-execution 解析（resolve_env_refs 不抛 ValueError）"
output_files:
  - "skills/topo-planning/src/exporter.py#IP 规划段"
file_ownership:
  primary:
    - "skills/topo-planning/src/exporter.py#IP 规划段"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "technical-note"
  trigger_reasons:
    - "复用既有 IPAM，模式既有"
  rationale: "接入点明确，Story 技术说明足够"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S6.md"
---

# STORY-CR037-S6：IP 规划集成

## 目标

在 `skills/topo-planning/src/exporter.py` 中增加 `compute_ip_plan(...)` 桥接函数：复用 topo-config 既有 `ipam.IPAM`（自动模式 198.18.0.0/15 按序分配 /24；preset 模式 per-broadcast-domain 指定）与 `topology_exporter` 的 `_compute_broadcast_domains`（含 DUT-DUT 直连 L3 域），产出 `ip_plan`（权威 schema = `ipam.ip_plan_to_dict`）供 S5 `export_env_file` 渲染 env-file 的 IP/gw/next_hop 字段。不新建 IPAM 引擎。**SW（L2）拓扑的广播域依赖 `vlan_assign`（`VlanIsolator` 产出）；`vlan_assign=None` 且拓扑含 SW 时显式报错，不得静默跳过 L2 域产出假成功 `ip_plan`。**

## 依赖

- S5（exporter.py 建立，`ip_plan` 接入点已预留；`allow_degraded` / `executable=false` 降级语义由 S5 承载）

## 输出文件

| 文件 | 说明 |
|------|------|
| `skills/topo-planning/src/exporter.py#IP 规划段` | 在 S5 exporter.py 内新增 `compute_ip_plan` 桥接段 |

## AI 任务清单

- [ ] TASK-S6-01：在 exporter.py 增加 `compute_ip_plan(mapping_result, logical_topo, pool, vlan_assign=None, preset_subnets=None) -> dict`
- [ ] TASK-S6-02：复用 `topology_exporter._compute_broadcast_domains` 计算广播域（含 L3 直连域）；含 SW 拓扑前置校验 `vlan_assign` 必传（None → `IP_PLAN_SW_VLAN_MISSING`）+ 广播域结果校验（缺 switch_ports 非空 L2 域 → `IP_PLAN_SW_L2_DOMAIN_MISSING`）
- [ ] TASK-S6-03：复用 `ipam.IPAM` 分配子网（interactive=False 自动 / preset 指定），容量校验
- [ ] TASK-S6-04：`ip_plan` 结构对齐 `ipam.ip_plan_to_dict` 权威 schema（subnet/gateway/endpoints，endpoints 不含 gateway 设备）
- [ ] TASK-S6-05：接入 S5 `_resolve_ip_plan`（nodes.*.interfaces.*.ip/gw + dut.next_hop）
- [ ] TASK-S6-06：单测：自动模式 / preset 模式 / 容量不足错误 / 广播域含 DUT-DUT L3 域 / SW 拓扑缺 vlan_assign 报错

## 技术说明

### 设计依据

- HLD §4.6：IP 规划（S6）复用 topo-config IPAM（198.18.0.0/15 自动分配 or per-broadcast-domain 指定），广播域按导出器计算（含 DUT-DUT 直连 L3 域）。
- 既有能力：`skills/topo-config/src/ipam.py` 提供 `IPAM.plan(broadcast_domains, interactive=False, preset_subnets=None)`（自动/预设两模式 + 容量校验）与 `ip_plan_to_dict`（权威输出 schema，ipam.py:222-235）；`skills/topo-config/src/topology_exporter.py` 提供 `_compute_broadcast_domains(mapping, topo, vlan_assign, pool)`（L2/L3 广播域计算，vlan_assign 为 None 时 L2 SW 域缺失——`_compute_l2_broadcast_domains` 的 `switch_ops` 循环为空，topology_exporter.py:107）。S6 只做桥接，不重复实现。
- **SW L2 域来源（评审 S6-A 修正）**：广播域计算必须消费 `vlan_assign`（`topo-config VlanIsolator` 产出，`VlanAssignment{env_name, vlan_id, switch_ops, skipped_endpoints}`，vlan_isolator.py:43-48）。含 SW（L2）节点的拓扑，`vlan_assign=None` 会使 SW 各 VLAN 广播域静默缺失 → 相关 TG/SW/DUT 链路 IP 不分配 → S5 渲染空 IP → `resolve_env_refs` 抛 ValueError（op_mapper.py:1123-1125）。因此：`vlan_assign=None` 且拓扑含 SW → 显式报错 `IP_PLAN_SW_VLAN_MISSING`；无 SW 拓扑可缺省 None（走 DUT-DUT 直连域逻辑）。调用链：S7 CLI / S8 编排先调用 `VlanIsolator`（或复用 `EnvironmentManager.create` Stage 4）生成 `vlan_assign`，再传入 `compute_ip_plan`。
- 与 S5 边界：S5 `export_env_file` 已预留 `ip_plan=None` 降级——但降级产物**必须**标记 `metadata.executable=false` + `degraded_reason`，默认缺 IP 直接报错（S5 承接），S6 交付 `compute_ip_plan` 后由 S7 CLI / S8 编排传入 `ip_plan`。

### 文件影响

| 文件 | 操作 | 说明 |
|------|------|------|
| `skills/topo-planning/src/exporter.py#IP 规划段` | **修改（S5 之上追加）** | 新增 `compute_ip_plan` 桥接函数 + `_resolve_ip_plan` 接入；import `ipam.IPAM` / `ipam.ip_plan_to_dict` 与 `topology_exporter._compute_broadcast_domains` |
| `skills/topo-config/src/ipam.py` | **不修改** | 只读复用（`IPAM.plan` / `ip_plan_to_dict` 权威 schema） |
| `skills/topo-config/src/topology_exporter.py` | **不修改** | 只读复用（`_compute_broadcast_domains`） |
| `skills/topo-config/src/vlan_isolator.py` | **不修改** | 只读消费 `VlanAssignment` 类型（SW L2 域来源；由 S7 CLI / S8 编排在调用前生成 vlan_assign） |
| `skills/case-execution/*` | **不修改** | 消费方不变 |

### 接口变化

- 新增函数：`compute_ip_plan(mapping_result, logical_topo, pool, vlan_assign=None, preset_subnets=None) -> dict`
  - `preset_subnets`：`{domain_id: subnet}` 可选；`None` → 自动模式（198.18.0.0/15 按序分配 /24）。
  - `vlan_assign`：`VlanAssignment` 或 None。**含 SW（L2）节点的拓扑必传**（None → 抛 `ExporterError(IP_PLAN_SW_VLAN_MISSING)`）；无 SW 拓扑可缺省 None。
  - 返回：`{domain_id: {subnet, gateway: {device, port, ip}, endpoints: [{device, port, ip, gateway, trex_port}]}}`（**权威 schema = `ipam.ip_plan_to_dict`**，ipam.py:222-235；endpoints 不含 gateway 设备，ip 带掩码，endpoint.gateway 无掩码）。
  - 广播域来源：`topology_exporter._compute_broadcast_domains(mapping, logical_topo, vlan_assign, pool)`；L3 直连（DUT-DUT p2p）域一并纳入。
  - **广播域结果校验**：若逻辑拓扑含 SW 且返回的 domains 中无任何 `switch_ports` 非空的 L2 域 → 抛 `ExporterError(IP_PLAN_SW_L2_DOMAIN_MISSING)`，防止 vlan_assign 传入但 L2 域仍静默缺失。
- 接入 S5：`_resolve_ip_plan(nodes, ip_plan, port_mapping)` 已由 S5 定义，S6 填充真实 `ip_plan` 后生效。
- 无 CLI 独立入口；由 S7 `topo-plan` CLI 或 S8 编排调用（先经 `VlanIsolator` 生成 `vlan_assign` 再传入）。

### 数据变化

- 无持久化 schema 变化：`ip_plan` 为内存 dict，仅渲染进 env-file；结构以 `ipam.ip_plan_to_dict` 为权威 schema（ipam.py:222-235）。
- `ipam.IPAM` 自动模式使用保留段 `198.18.0.0/15`（RFC 2544 benchmark 段），`/24` 前缀；池耗尽自动切 `100.64.0.0/10`（既有行为）。
- preset 模式：`{domain_id: subnet}` 校验子网合法 + 不重复；非法/重复报 `ValueError`。
- `vlan_assign` 为调用期内存对象（`VlanAssignment`），不落盘；SW L2 域完全由它驱动（`_compute_l2_broadcast_domains` 的 `switch_ops` 循环），不可省略。

### 权限变化

- 无新增权限。`compute_ip_plan` 纯内存计算，不涉及凭据/网络写操作。
- 复用 `topology_exporter` 只读计算；不触碰物理池/台账写接口。

### 异常、失败与回退

**失败路径**（结构化暴露，不静默吞错）：

- 容量不足：`ipam.IPAM.plan` 抛 `ValueError`（端点+网关 > 子网可用 IP）→ `compute_ip_plan` 转 `ExporterError(code="IP_CAPACITY_EXCEEDED", ...)`，由 S7 CLI / S8 显示结构化失败信息。
- preset 子网非法/重复：`ValueError`，提示合法子网格式，调用方可修正后重试。
- 广播域计算失败（`_compute_broadcast_domains` 输入契约不匹配）：`ExporterError(code="BROADCAST_DOMAIN_ERROR", ...)` + 适配建议。
- **SW（L2）拓扑缺 vlan_assign（评审 S6-A 强制）**：逻辑拓扑含 SW（L2）节点且 `vlan_assign=None` → `ExporterError(code="IP_PLAN_SW_VLAN_MISSING", "含 SW 的 L2 拓扑广播域依赖 vlan_assign（VlanIsolator 产出），禁止静默跳过 L2 域")`。
- **SW L2 域缺失校验**：逻辑拓扑含 SW 但 `_compute_broadcast_domains` 返回的 domains 无 `switch_ports` 非空的 L2 域 → `ExporterError(code="IP_PLAN_SW_L2_DOMAIN_MISSING", ...)`，防止假成功。

**回退**：

- 广播域为空（映射结果无可用 L2/L3 域，如无 SW 且无 DUT-DUT 直连）：返回空 `ip_plan`；S5 缺 IP 默认抛 `IP_PLAN_REQUIRED`，仅显式降级（dry-run / `--allow-degraded`）产出 `executable=false` 降级产物，不进入 case-execution 消费路径。
- S6 未交付时 S5 `ip_plan=None` 降级路径仍可用（`executable=false` 降级产物，非静默空 IP）。

### 测试入口

- 单测：`skills/topo-planning/tests/test_exporter.py` 扩展（或 `test_ip_planning.py`）：
  - 自动模式：fixture 映射 + 广播域 → `ip_plan` 子网位于 `198.18.0.0/15`，`/24` 前缀，无交互；结构对齐 `ipam.ip_plan_to_dict`（endpoints 不含 gateway 设备、ip 带掩码、endpoint.gateway 无掩码）。
  - preset 模式：`{bd_id: '198.18.10.0/24'}` → 使用指定子网。
  - 容量不足：端点 > 可用 IP → `ExporterError(IP_CAPACITY_EXCEEDED)`。
  - 广播域含 DUT-DUT L3 直连域（fixture `node2_dut1_tg1_link3`）。
  - **SW（L2）拓扑 + `vlan_assign=None`（评审 S6-A）**：`ExporterError(IP_PLAN_SW_VLAN_MISSING)`，不产出缺 L2 域的假成功 ip_plan；传 vlan_assign 后正常产出含 SW L2 域。
  - **SW L2 域缺失校验**：含 SW 拓扑但 domains 无 switch_ports 非空 L2 域 → `IP_PLAN_SW_L2_DOMAIN_MISSING`。
- 集成（S10）：`compute_ip_plan → export_env_file → case_runner resolve_env_refs` 校验 `${ENV.tg.port1.ip}` 等解析不抛 ValueError。
- 复用 P-1 fixture 池（hg3250 + TE 端口）。

### 风险与重访条件

| 风险 | 级别 | 重访条件 |
|------|------|---------|
| `topology_exporter._compute_broadcast_domains` 输入契约（vlan_assign/vrf_assign）与 S4 MappingResult 不完全匹配 | 中 | 接入时报参数缺失 → 需要适配层（读取 S4 `MappingResult` 转换），转为 S6 追加任务或回 S5 设计修订 |
| `ipam.IPAM.plan` 交互参数签名变化 | 低 | 以 `interactive=False` + `preset_subnets` 调用，签名稳定 |
| IP 段与目标项目既有网段冲突 | 低 | 使用 RFC 2544 保留段（既有默认），目标项目可用 preset 模式显式指定 |
| S5/S6 同文件（exporter.py）并行编辑冲突 | 中 | Wave 2 串行（S6 依赖 S5），CP4 已确认无冲突 |
| SW（L2）拓扑 `vlan_assign` 缺失导致 L2 域静默丢失（评审 S6-A） | 高 | 默认显式报错（`IP_PLAN_SW_VLAN_MISSING` / `IP_PLAN_SW_L2_DOMAIN_MISSING`），不产出假成功；S7 CLI / S8 编排必须先行接入 `VlanIsolator` 生成 `vlan_assign` |

### 偏离记录

- 相对 HLD §4.6 与 DEVELOPMENT-PLAN S6（technical-note，`exporter.py#IP 规划段`）：无功能偏离。
- **评审修订（CP5 NEEDS_REWORK 综合审核打回）**：S6-A 强制修正 `vlan_assign=None` 不得静默跳过 SW L2 域——`compute_ip_plan` 增加 `vlan_assign` 参数并做 SW 拓扑前置校验与广播域结果校验（`IP_PLAN_SW_VLAN_MISSING` / `IP_PLAN_SW_L2_DOMAIN_MISSING`），调用链要求 S7 CLI / S8 编排先经 `VlanIsolator` 生成 `vlan_assign`；`ip_plan` 结构明确以 `ipam.ip_plan_to_dict` 为权威 schema（对齐 S5 §8.3）。
