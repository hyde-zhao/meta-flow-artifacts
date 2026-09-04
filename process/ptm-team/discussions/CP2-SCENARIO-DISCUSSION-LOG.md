# CP2 场景讨论日志 — CR-038（PPPoE 链路规划能力）

> 用途：人类审计 + 中断恢复。下游正式消费以 `docs/product/USE-CASES-CR-038.md`、`REQUIREMENTS-CR-038.md`、`SCENARIOS-CR-038.yaml`、`TEST-MATRIX-CR-038.md`、`STORY-MAP-CR-038.md`、`MVP-SCOPE-CR-038.md` 为准，不以本日志为唯一输入。

- CR：CR-038
- 阶段：requirement-clarification（meta-pm 委托）
- 场景主体：`scenario_subject_type=target-artifact`（ptm-te topo/PPPoE 能力）
- 模式：standard
- 日期：2026-08-15

---

## 1. 阶段零调研结论（4 个 DQ）

### DQ-038-01 — PPPoE Server 建模

**调研事实**：
- `resource/network-topology/topology-collection.md:465` `node3_dut1_tg1_pppoe_link4` 将 PPPoE Server 建模为 `node_type: Mock, sub_type: pppoe-server, role: pppoe-server`（3 节点链式 tg1→dut1→pppoe→tg1）。
- `topo_mapper.py` `_is_mock()` 跳过 Mock 节点（`_build_candidates` 置 candidates=[]、`_build_result` 标 `status='virtual'`、含 Mock 端点 link 标 `unmapped` reason='含虚拟 Mock 端点'），Mock 不参与物理匹配。
- `physical_pool.yaml` 已预留 `node_type: SW` 节点（SW1/SW2 示例）+ `${DEVICE_TYPE_SW}` 占位，节点字段含 `management`（host/port/username/password）+ `interfaces`。
- 参考物理场景中 PPPoE Server 运行在真实 SW3（H3C 交换机），SW2/SW1 为 L2 透传。

**推荐：方案 (a) — 逻辑拓扑 `pppoe` 节点从 `Mock` 改为真实 `SW` 节点（`node_type: SW, role: pppoe-server, sub_type: pppoe-server`）**。

理由：
1. PPPoE Server 是运行在真实 SW 上的功能，需要映射到真实设备 + 下发配置，不是虚拟节点。
2. `physical_pool.yaml` 已支持 `node_type: SW`，`topo_mapper._build_candidates` 经 `pool.get_devices_by_type(['SW'])` 可直接匹配，无需新增匹配类型。
3. `role` 字段已存在于逻辑节点与 `DeviceMapping`，用 `role=pppoe-server` 区分普通 L2 透传 SW 与 PPPoE Server SW。
4. 避免在 topo_mapper 引入"Mock→SW 映射期替换"特殊路径，符合"最简方案"原则。

`Mock` 仍保留作为其它虚拟节点（PC 等）的建模方式，不删除。

### DQ-038-02 — env-file SW 契约

**调研事实**：
- `case-execution/SKILL.md:91-99` `${ENV.*}` 9 类占位符全为 tg/dut；`case_runner.build_env_topology` 硬编码 `tg1`/`dut1` + `port_mapping.portN` 的 `tg`/`dut` 双侧，仅校验 tg1/dut1 存在（不拒绝额外节点，向后兼容）。
- `exporter._validate_mvp_topology` 硬编码 `set(node_type_counts)=={'TG','DUT'}`；`_build_nodes` 硬编码 nodes={tg1, dut1}；`_build_port_mapping` 硬编码 tg/dut 双侧。
- CR-037 中 SW（L2）仅作透传，配置（VLAN/VRF）由 `switch_configurator` 从 `physical_pool` 取管理地址独立下发，不走 case-execution `${ENV.*}`。

**推荐：SW 进入 env-file `nodes`（`nodes.swN`，供追溯 + switch_configurator 寻址），**不**进入 `port_mapping`（保持 9 类 `${ENV.*}` 占位符不变），**不**新增 `${ENV.sw.*}` 占位符**。

理由：
1. case-execution 用例（TG 发流 + NGFW 拨号）不直接操作 SW；SW 的 PPPoE Server 配置由 topo-config `switch_configurator` 独立下发（从 physical_pool + 映射结果取 SW 管理地址），复用 CR-037 部署链路。
2. 保持 `${ENV.*}` 9 类占位符硬契约零适配，case_runner `build_env_topology` 仅校验 tg1/dut1，额外 `nodes.swN` 向后兼容。
3. SW 进入 env-file `links`（环回链路端点含 SW，可追溯）。
4. 若未来用例需在 case_steps 直接引用 SW 端口，再新增 `${ENV.sw.*}`（进 Deferred Ideas）。

### DQ-038-03 — PPPoE Client 归属

**调研事实**：
- `resource/factor-libraries/` 是测试**设计层**因子（变量/值域/样本，供 ptm-tde 生成测试用例），不执行设备配置；真实设备/端口/链路实例明确不进公共因子库（README §与拓扑实例的边界）。
- ptm-atomic 是**执行层**原子操作（fw_config_interface / fw_config_policy_route / tg_* 等），由 op_mapper 映射 + case_runner 编排执行。
- NGFW 侧 PPPoE 拨号（DUT2 拨号获 163.0.0.x）是运行时设备配置动作。

**推荐：归属 ptm-atomic 原子操作（执行层）**。新增 `fw_config_pppoe_client`（或复用/扩展 interface 族 pppoe mode）原子操作，op_mapper 增补 OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / ROLLBACK_STRATEGY 映射。factor-library 不承担执行职责；若未来需"PPPoE 拨号方式"等测试设计因子，属 ngfw-interface 或新建 ngfw-pppoe 库，与当前执行需求正交（进 Deferred Ideas）。

注意：ptm-atomic CLI 本体是否已暴露 PPPoE 拨号 op 需在 LLD 阶段核实（同 CR-033/037"不改 ptm-atomic 本体"边界）；若未暴露，需列为跨仓库依赖或经 op_mapper 映射到可用 op。

### DQ-038-04 — 真机授权

**确认：沿用 CR-037 DQ-037-04 独立 `runtime_authorization` 模式**。真机下发（H3C telnet PPPoE Server + NGFW web PPPoE Client + trex 发流验证）均为独立 runtime_authorization 决策项，不在 CP 批准中隐含。无需新决策类型。

---

## 2. Scenario Gray Areas（3-4 个）

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 推荐 | 状态 |
|---|---|---|---|---|---|
| SGA-01 | PPPoE 地址池动态分配如何与静态 IPAM 协调 | PPPoE 拨号获 163.0.0.x 是动态分配，与 CR-037 静态 IPAM（198.18.0.0/15 per-broadcast-domain）语义冲突；env-file 的静态 IP 规划不能覆盖 PPPoE 动态地址 | 范围/验证/IP 规划 | PPPoE Server 侧 ip pool 段（如 163.0.0.0/24）作为**配置参数**进入 PPPoE Server 配置，NGFW 侧 PPPoE 客户端接口 IP 由拨号动态获取，**不进 env-file 静态 IP 规划**；env-file 的 IPAM 仅覆盖 TG/DUT 的静态侧 | open（待 CP2 确认） |
| SGA-02 | SW 参与映射后的 IP/VLAN 归属 | PPPoE Server SW3 是 L3（GE0/1 bind virtual-template 需 L3 IP），SW2/SW1 是 L2 透传；IP 规划从"TG-DUT 直连 p2p"扩展到"含 SW 的 L2 域 + PPPoE L3 域" | 范围/复杂度/IPAM | 复用 exporter `compute_ip_plan` 已有 `_topology_has_sw` + `IP_PLAN_SW_VLAN_MISSING` 检查；PPPoE Server 的 L3 IP 作为 PPPoE 配置参数，L2 透传 SW 走 vlan_assign | open（待 CP2 确认） |
| SGA-03 | 环回链路方向性 | 参考场景"双向发流 0 丢包"，TG port1→port2 与 port2→port1 双向都要通；exporter `links` from/to 与 TG 发流 txport/rxport 的关系需明确 | 验证/交付 | env-file `links` 保持无向语义（端点对），TG 双向发流由用例 case_steps 的 txport/rxport 显式指定，不与 exporter links 方向耦合 | open（待 CP2 确认） |
| SGA-04 | PPPoE Server 是否复用现有 VRF 命令（`ip binding vpn-instance`） | 拨号场景是否需要 VRF 隔离，还是全局路由域；影响 PPPoE 命令族与 VRF 命令的交互 | 复杂度/命令模板 | 首版 PPPoE Server 在全局路由域（不强制 VRF），`pppoe-server bind virtual-template` + `ip pool` + `local-user`；VRF 绑定作为可选项，复用现有 `ip binding vpn-instance` | open（待 CP2 确认） |

---

## 3. 用户可见场景确认交互（SGQ）

### SGQ-038-01（SGA-01 + SGA-02 合并，最高价值）

**问题**：PPPoE 链路的 IP 规划与 env-file 承载——PPPoE 拨号地址（163.0.0.x 动态）与 SW 节点配置如何进入产物？

**候选选项**：
1. **（推荐 A）SW 配置与 PPPoE 地址池解耦**：SW 进入 env-file `nodes`（供 switch_configurator 寻址），PPPoE Server 配置（ip pool 段 + pppoe-server bind + local-user）由 topo-config `switch_configurator` 从 physical_pool + 映射结果独立下发；NGFW 侧 PPPoE 客户端接口 IP 动态获取，不进 env-file 静态 IPAM。env-file 静态 IPAM 仅覆盖 TG/DUT 静态侧。
2. **（B）SW 全量进 env-file**：SW 进 `nodes` + `port_mapping`，新增 `${ENV.sw.*}` 占位符，PPPoE 拨号地址池与 SW 端口均静态规划进 env-file，case-execution 可直接引用 SW。
3. **（C）SW 独立产物**：SW 配置走独立的 switch-config 产物文件（与 env-file 完全解耦），env-file 不含任何 SW 信息。

**推荐项**：A。理由：复用 CR-037 部署链路、保持 ${ENV.*} 9 类契约零适配、最简；PPPoE 动态地址语义天然属于"配置参数"而非"静态 IPAM"。

**影响面**：范围、IP 规划集成、验证场景、后续门控。

**用户回答**：待 CP2 确认（本轮未回）。
**复述确认**：待用户确认后回填。
**影响面结论**：待 CP2 确认后回填。

### SGQ-038-02（SGA-03）

**问题**：环回链路的 env-file `links` 方向性——TG 双向发流（port1→port2、port2→port1）是否需要 env-file 显式表达方向？

**候选选项**：
1. **（推荐 A）links 无向，方向由用例显式指定**：env-file `links` 保持端点对（无向），TG 双向发流方向由 case_steps 的 `tg_start_traffic_stream` txport/rxport 显式指定。
2. **（B）links 有向**：env-file `links` 新增 direction 字段，表达环回方向。

**推荐项**：A。理由：与 CR-037 env-file links 契约（from/to 无向端点对）一致，发流方向本就用例层关注点。

**用户回答**：待 CP2 确认。
**复述确认**：待用户确认后回填。

### SGQ-038-03（SGA-04）

**问题**：PPPoE Server 首版是否强制 VRF 隔离（复用现有 `ip binding vpn-instance`）？

**候选选项**：
1. **（推荐 A）首版全局路由域，VRF 可选**：PPPoE Server 默认全局路由域，VRF 绑定作为可选参数复用现有 VRF 命令，首版不强制。
2. **（B）首版强制 VRF**：PPPoE Server 必须绑定 vpn-instance。

**推荐项**：A。理由：参考场景未涉及 VRF 隔离，首版最小实现，VRF 可选避免过度设计。

**用户回答**：待 CP2 确认。
**复述确认**：待用户确认后回填。

---

## 4. Deferred Ideas

| ID | 想法/风险/扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-038-01 | `${ENV.sw.*}` 占位符（用例直接引用 SW 端口） | SGA/SGQ-01 选项 B | 当前参考场景用例不直接操作 SW，SW 配置走 topo-config 独立下发 | 未来用例需 case_steps 直接操作 SW 端口时 |
| DEF-038-02 | ngfw-pppoe factor-library（PPPoE 拨号方式等测试设计因子） | DQ-038-03 | PPPoE 拨号当前是执行动作，非测试设计变量 | 未来 ptm-tde 需生成 PPPoE 测试用例设计因子时 |
| DEF-038-03 | PPPoE Server 强制 VRF 隔离 | SGA-04 | 参考场景未涉及 VRF | 未来拨号场景需要 VRF 隔离时 |
| DEF-038-04 | SW 配置回滚闭环（teardown 撤销 pppoe-server bind / ip pool / local-user） | 覆盖扫描 | 首版只做下发 + 台账占用闭环，SW 配置回滚留待验证 | CP7 验证暴露回滚需求时 |

---

## 5. 讨论状态

- 本轮由 meta-pm 完成阶段零调研 + 场景发现 + 灰区识别，产出 `USE-CASES-CR-038.md` draft。
- 4 个 DQ 已有结论与推荐；3 条 SGQ 已记录（待 CP2 用户确认）。
- 恢复点：`process/checks/CP2-DISCUSSION-CHECKPOINT.json`。
- 待 host-orchestrator 发起 CP2 正式人工确认。
