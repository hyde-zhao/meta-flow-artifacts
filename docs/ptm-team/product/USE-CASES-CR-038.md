---
status: draft
version: "1.0"
confirmed_by: ""
confirmed_at: ""
engagement_mode: production
scenario_subject_type: target-artifact
scenario_subject_id: ptm-te-topo-pppoe
target_artifact_type: skill
governance_mode: conditional
review_policy: strict
delivery_routing:
  mode: project-readme-contract
  output_root: "skills/topo-config/, skills/topo-planning/, skills/device-management/, skills/case-execution/, agents/ptm-te.md, resource/network-topology/"
  source: user-confirmed
total_use_cases: 6
cr_id: CR-038
parent_cr: CR-037
---

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|------|------|--------|----------|--------------|
| 1.0 | 2026-08-15 | meta-pm | CR-038 初始场景基线（PPPoE 链路规划能力） | 初始化基线（新增文档，无既有 USE-CASES 基线可追溯） |
| 1.1 | 2026-08-15 | meta-pm | CP2 反馈修订：SM-038-02 改口径（H3C 真机验证 + ruijie/huawei dry-run diff）；UC-038-04 纳入多实例 TRex | 原文档更新 |

## 用户画像（Personas）

| 画像 ID | 角色名称 | 典型背景 | 核心诉求 | 技术水平 |
|---------|---------|---------|---------|---------|
| P-038-01 | 测试执行工程师 | 负责跑 PPPoE 环回链路测试用例，需把逻辑 topo 映射到真实 SW/NGFW/TG 并一键下发配置 | 用 ptm-te 生成 env-file 与 SW 配置，跑通 TG 双向发流 0 丢包验证 | 中级 |
| P-038-02 | 测试平台开发者 | 维护 ptm-te topo 引擎，扩展 SW 节点映射 / PPPoE 命令 / 环回 env-file 能力 | 引擎能力通用化，不固化单条链路，映射失败可结构化诊断 | 高级 |
| P-038-03 | 网络工程师 | 维护 SW 设备台账（H3C 等）与接线，提供 PPPoE Server 设备的真实接线/端口 | SW 设备 schema 与平台别名可配置，台账不被测试过程破坏 | 高级 |

## 成功指标（Success Metrics）

| 指标 ID | 指标名称 | 度量方式 | 目标值 |
|---------|---------|---------|--------|
| SM-038-01 | SW 显式节点映射成功率 | `node_type=SW` 逻辑节点在 physical_pool 有候选时完成映射 | 100%（含 role=pppoe-server 的 PPPoE Server SW） |
| SM-038-02 | PPPoE 命令厂商验证 | H3C 真机验证通过；ruijie/huawei 仅 dry-run diff 供人工核对 | H3C 真机验证通过 |
| SM-038-03 | 环回拓扑 env-file 导出 | exporter 突破 MVP 单 TG+单 DUT，导出含 SW 节点的环回 env-file | 参考场景 1 条环回链路可导出 |
| SM-038-04 | PPPoE 链路 0 丢包 | trex 双向发流 `tg_verify_traffic_loss` | loss=0 |

## 明确排除（Out of Scope）

- 固化单条 PPPoE 环回链路（交付形态是通用 topo 引擎能力，非单链路硬编码）。
- `${ENV.sw.*}` 占位符（SW 不进入 port_mapping，case-execution 不直接操作 SW，见 DEF-038-01）。
- ngfw-pppoe factor-library（PPPoE 拨号是执行动作，非测试设计因子，见 DEF-038-02）。
- PPPoE Server 强制 VRF 隔离（首版全局路由域，见 DEF-038-03）。
- SW 配置回滚闭环（teardown 撤销下发命令，留待 CP7 验证，见 DEF-038-04）。
- ptm-atomic CLI 本体修改（沿用 CR-033/037 边界，PPPoE op 暴露问题在 LLD 核实）。

## 治理附录（Governance）

| 字段 | 当前值 | 说明 |
|------|--------|------|
| `engagement_mode` | production | CR-038 是 ptm-te 目标能力交付，非 meta-flow 自改进 |
| `scenario_subject_type` | target-artifact | 场景主体是 ptm-te topo/PPPoE 能力 |
| `scenario_subject_id` | ptm-te-topo-pppoe | 当前场景真正服务的对象 |
| `target_artifact_type` | skill | 交付形态为 ptm-te skill 能力扩展 |
| `governance_mode` | conditional | 标准工作流，真机下发独立 runtime_authorization |
| `review_policy` | strict | PPPoE 命令/凭据/真机下发属高风险，严格审查 |
| `delivery_routing.mode` | project-readme-contract | 沿用 CR-037 交付出口 |
| `delivery_routing.output_root` | skills/*、agents/ptm-te.md、resource/network-topology/ | 已确认交付根目录 |
| `delivery_routing.source` | user-confirmed | 用户已拍板交付形态与 SW 纳入映射 |

## Scenario Gray Areas

**Discussion Log**：`process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md`
**Checkpoint**：`process/checks/CP2-DISCUSSION-CHECKPOINT.json`

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 | 状态 |
|---|---|---|---|---|---|
| SGA-01 | PPPoE 地址池动态分配如何与静态 IPAM 协调 | 拨号地址动态，与静态 IPAM 语义冲突 | 范围/验证/IP 规划 | 待 CP2（SGQ-038-01） | open |
| SGA-02 | SW 参与映射后的 IP/VLAN 归属 | SW3 是 L3（PPPoE Server），SW2/SW1 是 L2 透传 | 范围/复杂度/IPAM | 待 CP2（SGQ-038-01） | open |
| SGA-03 | 环回链路方向性 | 双向发流，links 无向 vs 有向 | 验证/交付 | 待 CP2（SGQ-038-02） | open |
| SGA-04 | PPPoE Server 是否复用现有 VRF 命令 | 拨号是否需要 VRF 隔离 | 复杂度/命令模板 | 待 CP2（SGQ-038-03） | open |

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-038-01 | `${ENV.sw.*}` 占位符 | SGQ-038-01 选项 B | 当前用例不直接操作 SW | 未来用例需 case_steps 引用 SW 端口时 |
| DEF-038-02 | ngfw-pppoe factor-library | DQ-038-03 | PPPoE 拨号是执行动作 | 未来需 PPPoE 测试设计因子时 |
| DEF-038-03 | PPPoE Server 强制 VRF | SGA-04 | 参考场景未涉及 VRF | 未来拨号需 VRF 隔离时 |
| DEF-038-04 | SW 配置回滚闭环 | 覆盖扫描 | 首版只做下发 + 台账占用闭环 | CP7 暴露回滚需求时 |

## 使用场景列表

### UC-038-01：SW 显式节点映射

| 字段 | 内容 |
|------|------|
| **使用角色** | P-038-01、P-038-02 |
| **触发条件** | 逻辑拓扑含 `node_type: SW` 节点（普通 L2 透传 SW，或 `role: pppoe-server` 的 PPPoE Server SW） |
| **输入** | 逻辑拓扑（Topology/NodeInfo，含 `node_type: SW` + `role` + `sub_type` + `interfaces`）+ physical_pool（`node_type: SW` 节点含 management + interfaces） |
| **处理逻辑** | topo_mapper 将 `node_type=SW` 纳入候选（`pool.get_devices_by_type(['SW'])`），按连通性/接口/role 约束匹配，突破仅 L2 透传；`role=pppoe-server` 标记 PPPoE Server SW；映射结果含 SW 节点（非 virtual） |
| **输出/结果** | 映射结果含 SW 节点的 device_mapping（status=matched）+ link_mapping；PPPoE Server SW 带 role 标记 |
| **前置条件** | physical_pool 存在 `node_type: SW` 节点；逻辑节点 `node_type=SW` 非 Mock |
| **排除情况** | Mock 虚拟节点（PC 等）仍按 `_is_mock` 跳过；不含 SW 的 CR-037 单 TG+单 DUT 场景保持原行为 |

**处理流程（文字描述）：**
1. 步骤一：识别逻辑拓扑中 `node_type=SW` 节点，与 Mock 节点区分（Mock 保持虚拟）。
2. 步骤二：`_build_candidates` 对 SW 节点经 `pool.get_devices_by_type(['SW'])` 生成候选。
3. 步骤三：按接口匹配 + 连通性约束 + `role`（pppoe-server）匹配真实 SW 设备。
4. 步骤四：生成 device_mapping（SW 节点 status=matched，含 role）+ link_mapping。

---

### UC-038-02：PPPoE Server 配置生成与下发

| 字段 | 内容 |
|------|------|
| **使用角色** | P-038-01、P-038-02、P-038-03 |
| **触发条件** | 映射结果存在 `role=pppoe-server` 的 SW 节点 |
| **输入** | physical_pool SW 节点（management host/port/username/password 占位）+ PPPoE Server 配置参数（ip pool 段、virtual-template 编号、local-user 名/密文密码、认证方式） |
| **处理逻辑** | switch_configurator 新增 `configure_pppoe_server`；commands 新增 PPPoE 命令族（三厂商）：`ip pool`、`pppoe-server bind virtual-template`、`virtual-template` 的 `ppp authentication-mode` / `remote address pool`、`local-user`（`service-type ppp` + cipher 密文密码）；H3C 为参考实现 |
| **输出/结果** | SW 配置命令序列（dry-run 输出 diff，真机下发需 runtime_authorization）+ 台账占用闭环 |
| **前置条件** | SW 节点已完成映射；PPPoE Server 参数已给出；默认 dry-run |
| **排除情况** | 强制 VRF 隔离（首版全局路由域，VRF 绑定可选）；真机下发未获独立 runtime_authorization 前禁止 |

**处理流程（文字描述）：**
1. 步骤一：识别 `role=pppoe-server` SW 节点及其 management 地址。
2. 步骤二：生成 ip pool + pppoe-server bind virtual-template + virtual-template + local-user 命令序列。
3. 步骤三：dry-run 输出 diff；真机下发前校验 runtime_authorization。
4. 步骤四：写台账占用，保证 PPPoE Server 端口/地址池不冲突。

---

### UC-038-03：PPPoE Client 配置（NGFW 拨号）

| 字段 | 内容 |
|------|------|
| **使用角色** | P-038-01、P-038-02 |
| **触发条件** | PPPoE 链路中 NGFW（DUT）作为 PPPoE Client，需拨号获 163.0.0.x |
| **输入** | NGFW 接口（TE0_8）PPPoE 拨号参数（user/password 占位、拨号接口、地址获取方式） |
| **处理逻辑** | ptm-atomic 原子操作（执行层）新增 `fw_config_pppoe_client`（或扩展 interface 族 pppoe mode），op_mapper 增补 OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / ROLLBACK_STRATEGY 映射，case_runner 编排执行 |
| **输出/结果** | NGFW 侧 PPPoE Client 拨号配置完成，接口动态获 163.0.0.x |
| **前置条件** | ptm-atomic CLI 暴露 PPPoE 拨号 op（LLD 阶段核实）；凭据占位 |
| **排除情况** | factor-library 设计层因子（PPPoE 拨号是执行动作，非设计变量） |

**处理流程（文字描述）：**
1. 步骤一：定位 NGFW 拨号接口 TE0_8 与 PPPoE 参数。
2. 步骤二：经 ptm-atomic op（fw_config_pppoe_client）下发拨号配置。
3. 步骤三：校验拨号状态（接口获取 163.0.0.x）。
4. 步骤四：进入双向发流验证（UC-038-06）。

---

### UC-038-04：环回拓扑 env-file 导出

| 字段 | 内容 |
|------|------|
| **使用角色** | P-038-01、P-038-02 |
| **触发条件** | 逻辑拓扑为 TG+DUT+SW 多节点 + 环回链路 |
| **输入** | 映射结果（含 SW 节点）+ IP 规划结果 |
| **处理逻辑** | exporter 突破 `_validate_mvp_topology`（单 TG+单 DUT 硬编码），支持 TG+DUT+SW 多节点 + 环回；`_build_nodes` 扩展 SW 节点（`nodes.swN` 含 host/brand/role/interfaces），`links` 表达环回端点对（无向）；SW 不进入 port_mapping |
| **输出/结果** | env-file（nodes 含 tg1/dut1/sw*，port_mapping 保持 tg/dut 双侧，links 含环回） |
| **前置条件** | 映射结果含 SW；IP 规划完成 |
| **排除情况** | `${ENV.sw.*}` 占位符（SW 不进 port_mapping）；links 方向性显式字段（发流方向由用例指定） |

**处理流程（文字描述）：**
1. 步骤一：放宽 `_validate_mvp_topology`，识别多节点 + 环回。
2. 步骤二：`_build_nodes` 输出 tg1/dut1/sw* 节点。
3. 步骤三：`_build_port_mapping` 保持 tg/dut 双侧；`_build_links` 表达环回端点对。
4. 步骤四：写 env-file，校验 `load_env_file` 兼容（额外 sw 节点不破坏 tg1/dut1）。

---

### UC-038-05：SW 设备 schema 与平台别名

| 字段 | 内容 |
|------|------|
| **使用角色** | P-038-03 |
| **触发条件** | SW 设备需在台账中声明（devices.yaml）与平台别名映射（device-reference） |
| **输入** | SW 设备信息（host/port/user/password 占位 + brand，如 H3C） |
| **处理逻辑** | devices.yaml 新增 `device_groups.sw` 块；device-reference.md 新增 SW 硬件平台别名映射（H3C 等，唯一真相源） |
| **输出/结果** | SW 设备 schema + 平台别名映射可用，topo-config 可据此寻址下发 |
| **前置条件** | SW 设备真实接线/端口已知 |
| **排除情况** | 凭据明文（一律 `${ENV_VAR}` 占位，ADR-02） |

**处理流程（文字描述）：**
1. 步骤一：devices.yaml 新增 sw 设备块 schema（host/port/user/password/brand）。
2. 步骤二：device-reference 新增 SW 平台别名映射（H3C → 厂商命令族）。
3. 步骤三：验证 topo-config 能经 devices.yaml 寻址 SW。

---

### UC-038-06：PPPoE 链路双向发流验证

| 字段 | 内容 |
|------|------|
| **使用角色** | P-038-01 |
| **触发条件** | PPPoE Server/Client 配置完成，环回拓扑 env-file 就绪 |
| **输入** | env-file + TG 发流参数（txport/rxport + 报文模板） |
| **处理逻辑** | trex-traffic：acquire/config/apply/start → 双向发流 → `tg_verify_traffic_loss` 校验 loss=0 → stop/delete/release |
| **输出/结果** | 双向发流 0 丢包验证结果 |
| **前置条件** | PPPoE Server/Client 配置已下发；trex-api 可用；真机验证获 runtime_authorization |
| **排除情况** | 单向发流（参考场景要求双向）；未获真机授权前禁止真实发流 |

**处理流程（文字描述）：**
1. 步骤一：加载 env-file，构建 topo。
2. 步骤二：TG 配置双向发流流（port1→port2 与 port2→port1）。
3. 步骤三：apply/start 发流，`tg_verify_traffic_loss` 校验 0 丢包。
4. 步骤四：stop/delete/release 清理。

---

<!-- coverage-checklist: begin -->
## 附录：覆盖自检表

> 作为正式兼容结构的可见附录；仅记录覆盖状态，不改变正文必填字段集。

| 维度 ID | 维度名称 | 状态 | 涉及场景 | 备注 |
|---------|---------|------|---------|------|
| D1 | 用户维度 | 已覆盖 | UC-038-01~06 | P-038-01/02/03 三画像 |
| D2 | 任务维度 | 已覆盖 | UC-038-01~06 | 映射/配置/导出/验证 |
| D3 | 动机维度 | 已覆盖 | 全部 | SM-038-01~04 可度量 |
| D4 | 时间维度 | 已覆盖 | UC-038-02/03/06 | dry-run→真机授权→发流验证顺序 |
| D5 | 环境维度 | 已覆盖 | UC-038-01/02/05 | physical_pool/devices.yaml 三层文件 |
| D6 | 方式维度 | 已覆盖 | UC-038-02/03 | dry-run 默认，真机独立授权 |
| D7 | 异常维度 | 已覆盖 | UC-038-01/02 | 映射失败/无候选/真机未授权结构化报告 |
| D8 | 集成维度 | 已覆盖 | UC-038-03/06 | op_mapper/trex-traffic/case_runner 集成 |
| Dx | 自定义维度（凭据安全） | 已覆盖 | UC-038-02/03/05 | ${ENV_VAR} 占位 + cipher 密文 |
<!-- coverage-checklist: end -->
