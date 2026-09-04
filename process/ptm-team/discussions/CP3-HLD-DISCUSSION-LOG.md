# CP3-HLD 架构灰区讨论日志 — CR-038

- CR：CR-038（PPPoE 链路规划能力）
- 阶段：solution-design（meta-se 委托）
- 生成时间：2026-08-15
- 前置：CP2-CR038 已 approve（7 项决策不可推翻，作为 HLD 前提）
- 结论分类：本轮 4 个架构灰区全部归 **agent 默认处理**（实现级确定性决策），无新增需用户拍板的范围/安全/外部接口/权限类决策，不触发 AskUserQuestion。

## 灰区分类总览

| 灰区 ID | 主题 | 决策级别 | 结果 |
|---|---|---|---|
| AGA-038-01 | 显式 SW 节点 vs 自动 L2 透传的优先级与互斥（R-F-018 地基） | agent 默认处理 | 已确定性定义（reserved 集互斥） |
| AGA-038-02 | 多实例 TRex 数据契约（physical_pool + pool_merge + exporter 三层） | agent 默认处理 | 已确定性定义（pool 侧为准按 name 合并） |
| AGA-038-03 | PPPoE 数据源 schema + 命令族（physical_pool 新字段 + 三厂商命令） | agent 默认处理 | 已确定性定义（physical_pool `pppoe_server` 块） |
| AGA-038-04 | 环回拓扑 env-file 放行规则（_validate_mvp_topology 放宽 + links 校验 + interface_kind） | agent 默认处理 | 已确定性定义（1TG+1DUT+N SW） |

> 说明：CP2 已锁定的 7 项决策（DQ-038-01/02/03/05 + SGQ-038-01/02/03）覆盖了所有方向性/范围/架构决策；本 CP3 灰区均为这些决策在实现层的确定性落地，不引入新的用户决策项。

---

## AGA-038-01：显式 SW 节点 vs 自动 L2 透传的优先级与互斥

**背景**：CR-037 topo_mapper 仅通过 `_try_l2_pass_through` 在「无直连 cable 时」自动插入 L2 SW 透传路径。CR-038 引入显式 `node_type=SW` 逻辑节点（参考场景 SW1/SW2/SW3 均为显式节点）。两者若并存，同一物理 SW 可能被「显式映射」和「自动透传」双重占用，造成台账冲突（R-F-018 评审 #1 地基）。

**现状**：`_try_l2_pass_through` 已按 `assigned_sws`（回溯当前已分配的 SW）跳过；但该判定依赖回溯顺序——若透传 link 先于显式 SW 节点处理，显式 SW 可能被自动透传「抢先占用」。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 预计算 reserved 集（显式 SW 候选集整体从自动透传候选池剔除） | 与回溯顺序无关，确定性最强；彻底消除台账冲突；实现最小（一个预计算 + 一个跳过条件） | 若显式 SW 匹配失败，其候选集仍被保守保留、不降级给透传（略保守） | topo_mapper `_backtrack`/`_try_l2_pass_through`；台账占用 | **推荐（agent 默认）** | 假设显式 SW 节点优先于透传（R-F-018 已定）；若未来需「显式 SW 失败后透传复用其候选」再改为仅跳过已分配 SW |
| B. 仅跳过已分配 SW（维持现状 + 排序保证） | 改动最小 | 依赖回溯顺序，非确定性；顺序微调即破坏互斥 | topo_mapper 排序 | 不推荐 | 仅当显式 SW 与透传从不同拓扑出现时可用 |
| C. 显式 SW 强制前置阶段（先映射 SW 再映射 TG/DUT） | 语义清晰 | 打乱现有回溯单通道结构，改动面大 | topo_mapper 全流程 | 不推荐 | 仅当 reserved 集方案被证明不足时 |

**确定性算法（写入 HLD §4.1）**：
1. 回溯前预计算 `explicit_sw_reserved = { phys | phys 为任一显式逻辑 SW 节点的物理候选 }`。
2. `_try_l2_pass_through` 候选 SW 集合 = 全部 SW − `explicit_sw_reserved` − `assigned_sws`（已分配 SW）。
3. 显式 SW 节点按既有 `_backtrack` 正常匹配（node_type=SW 候选），匹配成功保留 `role`，走 `_commit_allocation` 写台账。
4. 台账互斥由 reserved 集保证：显式 SW 候选永不进入自动透传。

---

## AGA-038-02：多实例 TRex 数据契约（三层）

**背景**：参考场景 TG1 主机并跑 GE(:8000)/TE(:8001) 两 TRex 实例。数据需要从 physical_pool → pool_merge → exporter 三层透传 `trex_instances` 汇总块 + 接口级 `instance`，并保持 node2 单值向后兼容。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. physical_pool `trex_instances` 汇总块 + 接口级 `instance`，pool 侧为准按 name 合并 | 与台账同源（DQ-038-05 精神一致）；向后兼容（无块时回退单值）；结构自描述 | 新增 `trex_instances`/`instance` 字段 | physical_pool.py + pool_merge.py + exporter.py | **推荐（agent 默认）** | 假设 TG 实例由 physical_pool 台账定义；若实例改由 devices.yaml 唯一定义则调整合并方向 |
| B. 实例信息仅存 devices.yaml `tg.instances` | 复用 devices 侧 | 破坏「台账同源」，与 DQ-038-05 冲突 | pool_merge | 不推荐 | 仅当 physical_pool 不承载 TG 实例时 |
| C. 每实例独立 TG 节点（GE/TE 拆两节点） | 概念清晰 | 破坏「一台 TG 主机」语义，env-file tg1 单节点契约被破坏 | exporter/case_runner | 不推荐 | 仅当两实例分属不同物理主机时 |

**确定性契约（写入 HLD §4.3）**：
- physical_pool.yaml TG 节点：`trex_instances: {GE: {api_server, hardware_platform, sync_port?, async_port?}, TE: {...}}`；接口级 `interfaces.portN.instance: GE|TE`。
- pool_merge：`UnifiedNode.trex_instances` + `UnifiedInterface.instance`；`_merge_tg` 以 pool 侧 `trex_instances` 为准，按 name 补齐 devices 侧 `tg.instances[*]` 的 `port_map/ports/service`；新增 `_resolve_env(value)` 解析 `${TG_HARDWARE_PLATFORM}`/`${TG_API_SERVER}`，解析不到保留原串 + warning。
- exporter：`_resolve_tg_instance(tg_phys, logical_tg)`——逻辑 tg1 声明 `trex_instance` 时按名取实例，否则回退 `management.api_server`（node2 向后兼容）；tg1 节点产出 `trex_instance`/`trex_sync_port`/`trex_async_port` + 保留 `trex_api_url`。

---

## AGA-038-03：PPPoE 数据源 schema + 命令族

**背景**：DQ-038-05 已定 ip pool 段 + local-user 数据源 = physical_pool 新字段（与设备台账同源）。需确定确切 schema 与三厂商命令模板结构。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. physical_pool SW 节点 `pppoe_server` 块（ip_pool + local_users + virtual_template） | 台账同源（DQ-038-05）；与设备强绑定；单源校验简单 | SW 模板新增一块 | physical_pool.yaml + switch_configurator + commands | **推荐（agent 默认）** | 假设 ip pool 段与特定 SW 强绑定；若跨设备共享改独立配置（DQ-038-05 切换条件） |
| B. devices.yaml 独立 `pppoe` 块 | 复用 devices 侧 | 与设备台账分离，破坏「同源一致」 | devices.yaml | 不推荐 | 仅当 ip pool 段跨设备共享 |
| C. 独立 pppoe.yaml 配置 | 单一职责 | 引入第三产物面，与 MVP 最简原则冲突 | 新文件 | 不推荐 | 仅当多 SW 共享 pool 段 |

**确定性契约（写入 HLD §4.4）**：
- physical_pool.yaml SW 节点新增 `pppoe_server` 块：`ip_pool_name / ip_pool_subnet / ip_pool_gateway? / virtual_template / authentication(pap|chap|mschap) / local_users[{username, password_cipher, service_type}]`。
- `password_cipher` 必须 `${ENV_VAR}` 占位（ADR-02），cipher 密文。
- 同源校验点：`ip_pool_subnet` 是唯一拨号网段来源（与 NGFW 拨号 163.0.0.x 同源），不在 devices.yaml/独立配置重复定义；R-F-019 单测断言「唯一源 + 同源一致」。
- commands.py 三厂商新增 pppoe 命令族（H3C 参考实现，ruijie/huawei dry-run diff 供核对）；switch_configurator 新增 `configure_pppoe_server`。

---

## AGA-038-04：环回拓扑 env-file 放行规则

**背景**：CR-037 `_validate_mvp_topology` 硬编码「恰好 1 TG + 1 DUT，无 SW/Mock/PC」。CR-038 需放宽到「1 TG + 1 DUT + N SW」环回，同时保证 links 去重/自环/闭环完整性（SCN-038-BND-004）与 interface_kind 透传。

| Option | Props | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 放宽为「恰好 1 TG + 1 DUT + 任意 N SW」，Mock/PC/多 DUT 仍拒绝 | 最小放行，与参考场景（单 DUT）一致；守住 env-file 静态 IPAM 语义 | 多 DUT 仍不支持（本 CR 范围外） | exporter `_validate_mvp_topology` | **推荐（agent 默认）** | 假设参考场景单 DUT；多 DUT 拓扑后续 CR 扩展 |
| B. 任意节点组合全放行 | 通用 | 破坏 port_mapping/nodes 静态契约，组合爆炸（RA-038-004） | exporter 全链路 | 不推荐 | 仅当明确多 DUT 需求时 |
| C. 维持硬编码 + 新增独立环回导出器 | 不碰旧路径 | 双导出器重复，违背最简 | exporter | 不推荐 | 仅当放宽破坏 CR-037 回归时 |

**确定性契约（写入 HLD §4.5）**：
- `_validate_mvp_topology` 放宽：允许 node_type ∈ {TG, DUT, SW}；恰好 1 TG + 1 DUT；SW 任意数量；Mock/PC 仍拒绝。
- links 校验：① 去重（a-b 与 b-a 同端点对）；② 自环检测（a==b 报错）；③ 闭环完整性（每个非 Mock 节点 degree≥2 且图单连通分量）。
- SW 进 `nodes.swN`（host/brand/role/interfaces），不进 port_mapping（R-F-007/R-C-005）。
- `interface_kind` 透传：逻辑拓扑 dut1 接口声明 `interface_kind=pppoe` 时不产 ip、next_hop 保持 None（R-F-016/017），默认 physical 时按原逻辑。

---

## 结论

4 个灰区全部通过 advisor table 收敛为 agent 默认的确定性契约，无新增需用户拍板决策项。讨论结果已回写 HLD §4 与 ARCHITECTURE-DECISION-CR-038.md。

- 恢复点：`process/checks/CP3-DISCUSSION-CHECKPOINT.json`
- 正式产物：`docs/design/HLD-CR-038.md`、`docs/design/ARCHITECTURE-DECISION-CR-038.md`
