---
status: draft
version: "1.1"
confirmed_by: ""
confirmed_at: ""
ready_for_design: false
source_use_cases: [UC-038-01, UC-038-02, UC-038-03, UC-038-04, UC-038-05, UC-038-06]
cr_id: CR-038
parent_cr: CR-037
---

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|------|------|--------|----------|--------------|
| 1.0 | 2026-08-15 | meta-pm | CR-038 初始需求基线（PPPoE 链路规划能力） | 初始化基线（新增文档） |
| 1.1 | 2026-08-15 | meta-pm | CP2 反馈修订：新增多实例 TRex（R-F-013~017）、显式 SW 协调（R-F-018）、数据源契约（R-F-019）；SM-038-02 改口径；补 RA-038-007 | 原文档更新 |
| 1.2 | 2026-08-17 | host-orchestrator | RF009 回写：R-F-009 块名 `device_groups.sw`→`device_groups.switch`，字段集改为 host/role/device_type/hardware_platform；M3 里程碑同步 | 原文档更新 |

## 功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源场景 |
|----|---------|--------|---------|---------|
| R-F-001 | topo_mapper 支持 `node_type=SW` 作为显式逻辑节点参与物理匹配（突破仅 L2 透传） | P0 | Given 逻辑拓扑含 node_type=SW 节点 When 执行映射且 physical_pool 有 SW 候选 Then 映射结果含 SW 节点 device_mapping（status=matched） | UC-038-01 |
| R-F-002 | topo_mapper 识别 `role=pppoe-server` 的 SW 节点为 PPPoE Server SW，并在映射结果保留 role 标记 | P0 | Given 逻辑节点 node_type=SW 且 role=pppoe-server When 映射 Then device_mapping 保留 role=pppoe-server 标记 | UC-038-01 |
| R-F-003 | switch_configurator 新增 PPPoE Server 命令族（`ip pool` / `pppoe-server bind virtual-template` / `virtual-template`（`ppp authentication-mode` + `remote address pool`）/ `local-user`（`service-type ppp` + cipher 密文密码）） | P0 | Given role=pppoe-server SW 节点 + ip pool 段等参数 When 生成配置 Then 输出含完整 PPPoE Server 命令序列（H3C 参考实现） | UC-038-02 |
| R-F-004 | commands.py 新增 PPPoE 命令模板，覆盖三厂商（H3C 参考 / ruijie / huawei） | P1 | Given 厂商=h3c/ruijie/huawei When 渲染 PPPoE 命令 Then 三厂商均产出对应命令模板 | UC-038-02 |
| R-F-005 | PPPoE Client 拨号归属 ptm-atomic 原子操作（执行层）：新增/复用 `fw_config_pppoe_client` op，op_mapper 增补映射 | P0 | Given NGFW TE0_8 PPPoE 拨号参数 When 经 ptm-atomic 下发 Then NGFW 接口拨号获 163.0.0.x | UC-038-03 |
| R-F-006 | exporter 突破 `_validate_mvp_topology` 单 TG+单 DUT 硬编码，支持 TG+DUT+SW 多节点 + 环回链路 | P0 | Given TG+DUT+SW 环回拓扑 When 导出 env-file Then 不再报 TOPO_NODE_COUNT_UNSUPPORTED，env-file 含环回 links | UC-038-04 |
| R-F-007 | exporter `_build_nodes` 扩展 SW 节点（`nodes.swN` 含 host/brand/role/interfaces），SW 不进入 port_mapping | P1 | Given 映射结果含 SW When 导出 Then env-file nodes 含 sw*，port_mapping 保持 tg/dut 双侧 | UC-038-04 |
| R-F-008 | case_runner `load_env_file` 兼容额外 SW 节点（不破坏 tg1/dut1 校验） | P1 | Given env-file 含 nodes.swN When load_env_file 执行 Then 不报错且 tg1/dut1 校验通过 | UC-038-04 |
| R-F-009 | devices.yaml 新增 `device_groups.switch` 块（host/role/device_type/hardware_platform） | P1 | Given SW 设备信息 When 声明于 devices.yaml Then topo-config 可寻址 SW | UC-038-05 |
| R-F-010 | device-reference.md 新增 SW 硬件平台别名映射（H3C 等，唯一真相源） | P1 | Given SW 品牌=h3c When 平台别名解析 Then 映射到 H3C 命令族 | UC-038-05 |
| R-F-011 | pool_merge 支持 SW 节点归并与台账映射 | P1 | Given physical_pool SW 节点 When 归并 Then SW 台账映射一致无冲突 | UC-038-01 |
| R-F-012 | PPPoE 链路双向发流验证（trex-traffic，loss=0） | P0 | Given PPPoE Server/Client 配置完成 When TG 双向发流 Then `tg_verify_traffic_loss` loss=0 | UC-038-06 |
| R-F-013 | physical_pool.py 消费多实例 TRex 字段：`get_trex_instances(device_name)` 返回实例汇总 dict；`get_free_ports` 透传 `instance` + `trex_port`；`get_trex_instance_of_port(device_name, port_id)` 按端口反查所属实例 | P0 | Given topology 源数据含 `trex_instances` 汇总块 + 接口级 `instance` 归属 When 调用 physical_pool 接口 Then 返回实例汇总 / 端口透传 instance+trex_port / 按端口反查实例 | UC-038-04 |
| R-F-014 | pool_merge.py 归并多实例 TRex：`UnifiedNode` 增 `trex_instances` 字段；`UnifiedInterface` 增 `instance` 字段；`_merge_tg` 合并 pool 侧 `trex_instances` 与 devices 侧 `tg.instances`（按 name 合并，pool 侧为准补 devices 侧 port_map/ports/service）；新增 `_resolve_env(value)` 解析 `${TG_HARDWARE_PLATFORM}` / `${TG_API_SERVER}`（解析不到保留原串 + warning） | P0 | Given pool 侧 `trex_instances` + devices 侧 `tg.instances` When `_merge_tg` 归并 Then 按 name 合并且 pool 侧为准补 devices 侧；`_resolve_env` 解析占位符或保留原串 + warning | UC-038-04 |
| R-F-015 | exporter.py 导出多实例 TRex：tg1 节点产出 `trex_instance` / `trex_sync_port` / `trex_async_port` + 保留 `trex_api_url`；新增 `_resolve_tg_instance(tg_phys)`（逻辑拓扑 tg1 声明 `trex_instance` 时按名取实例，否则回退单值 `management.api_server`，向后兼容 node2 系列） | P0 | Given tg1 逻辑声明 `trex_instance` When 导出 Then 产出三字段 + trex_api_url；未声明则回退单值向后兼容 | UC-038-04 |
| R-F-016 | exporter dut1 接口 `interface_kind` 透传（默认 `physical`）；`interface_kind: pppoe` 接口不产 `ip` 字段（动态分配） | P0 | Given 逻辑拓扑 dut1 接口 `interface_kind=pppoe` When 导出 Then 该接口不产 ip 字段；默认 physical 时按原逻辑产 ip | UC-038-04 |
| R-F-017 | dut1 存在 `interface_kind: pppoe` 接口时 `next_hop` 保持 None | P1 | Given dut1 含 `interface_kind=pppoe` 接口 When 导出 env-file Then next_hop=null | UC-038-04 |
| R-F-018 | 显式 SW 节点优先级高于自动透传：已显式映射的 SW 不参与 `_find_l2_passthrough` / `_find_l2_switch_pass_through` 自动透传；互斥逻辑必须进 HLD（Story 拆解前定义，避免台账冲突） | P0 | Given 逻辑拓扑同时含显式 SW 节点与自动透传路径 When 映射 Then 显式 SW 优先匹配，已显式映射 SW 不参与自动透传 | UC-038-01 |
| R-F-019 | PPPoE 地址池网段 + local-user 用户名/cipher 密文数据源契约：明确 `ip pool` 网段、认证账号从哪个文件读（physical_pool 新字段 / devices.yaml / 独立配置），且与 NGFW 拨号获 `163.0.0.x` 同源一致 | P0 | Given PPPoE Server 配置生成 When 读取 ip pool 网段 + local-user Then 数据源唯一且与 NGFW 拨号地址同源一致 | UC-038-02 |

## 约束需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|----|---------|--------|---------|------|
| R-C-001 | 凭据一律 `${ENV_VAR}` 占位，禁止明文（ADR-02） | P0 | Given 配置/产物含凭据 Then 均为 ${ENV_VAR} 占位无明文 | UC-038-02/03/05 排除情况 |
| R-C-002 | local-user 密码保持设备 cipher 密文 | P0 | Given PPPoE local-user 密码 Then 以 cipher 密文呈现 | UC-038-02 排除情况 |
| R-C-003 | 默认 `--dry-run`；真机下发（H3C telnet / NGFW web / trex）为独立 runtime_authorization | P0 | Given 未获 runtime_authorization Then 禁止真机下发，仅 dry-run | UC-038-02/03/06 排除情况 |
| R-C-004 | 现有 GE 实例（GE1_1~4）禁止改动其进程/cfg/wrapper | P0 | Given 交付过程 Then 不触碰 GE1_1~4 实例 | CR-038 安全约束 |
| R-C-005 | SW 不进入 port_mapping，不新增 `${ENV.sw.*}` 占位符 | P1 | Given env-file 导出 Then port_mapping 保持 tg/dut 双侧，无 sw 占位符 | UC-038-04 排除情况 |
| R-C-006 | PPPoE 拨号地址（163.0.0.x 动态）不进 env-file 静态 IPAM；ip pool 段作为配置参数进入 PPPoE Server 配置，数据源契约见 R-F-019 | P1 | Given PPPoE 链路 Then env-file 静态 IPAM 仅覆盖 TG/DUT 静态侧，ip pool 段与 NGFW 拨号地址同源一致 | SGA-01 |

## 非功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|----|---------|--------|---------|------|
| R-NF-001 | PPPoE 命令厂商验证口径（H3C 真机验证，ruijie/huawei 仅 dry-run diff 供人工核对） | P1 | Given 三厂商 When 验证 Then H3C 真机验证通过，ruijie/huawei dry-run diff 供人工核对 | UC-038-02 前置条件 |
| R-NF-002 | 映射失败结构化报告（SW 无候选/接口不匹配/连通性冲突给出 reason） | P1 | Given SW 映射失败 Then 输出结构化 reason 而非静默跳过 | UC-038-01 排除情况 |
| R-NF-003 | SW 配置下发幂等（重复 dry-run 不重复占用台账） | P2 | Given 重复下发 Then 台账占用不冲突 | UC-038-02 前置条件 |
| R-NF-004 | 现有 CR-037 单 TG+单 DUT 场景回归无破坏 | P0 | Given CR-037 拓扑 When 映射/导出 Then 原行为不变 | UC-038-01 排除情况 |

## 风险与假设

| ID | 类型 | 内容 | 关联需求 | 缓解措施 |
|----|------|------|---------|---------|
| RA-038-001 | RISK | ptm-atomic CLI 本体未暴露 PPPoE 拨号 op，需跨仓库新增 | R-F-005 | LLD 阶段核实 op 可用性；若缺失列为跨仓库依赖或经 op_mapper 映射到可用 op |
| RA-038-002 | RISK | H3C 真机 PPPoE Server 命令与参考命令族存在厂商差异 | R-F-003/004 | H3C 为参考实现，dry-run 输出 diff 供人工核对，真机下发独立授权 |
| RA-038-003 | RISK | PPPoE 动态地址池与静态 IPAM 集成语义冲突 | R-C-006 | ip pool 段作为配置参数，动态地址不进静态 IPAM（SGA-01，待 CP2） |
| RA-038-004 | RISK | 环回拓扑放宽 MVP 边界引入多节点/环回组合爆炸 | R-F-006 | 首版覆盖参考场景 + 通用节点类型判定，不穷举组合 |
| RA-038-005 | ASSUMPTION | physical_pool SW 节点（SW1/SW2）已预留且真实接线可用 | R-F-001 | 阶段零调研确认 node_type=SW 已预留；PPPoE Server SW3 真实接线由 P-038-03 提供 |
| RA-038-006 | RESOLVED | 参考场景端口 DUT2 拨号口统一为 TE0_8（原 TE0_7 为笔误） | R-F-005 | 已统一为 TE0_8，参考场景正文与需求理解框架一致 |
| RA-038-007 | RISK | 真机下发 × 无 teardown 撤销：SW 配置回滚闭环（DEF-038-04）延后，未来真机下发无自动撤销路径 | R-C-003 | 登记风险接受；真机下发前人工核对 diff + 记录下发命令供手工回滚；CP7 暴露回滚需求时升级 |

## 里程碑建议

| 里程碑 | 包含需求 | 交付物 | 前置里程碑 |
|--------|---------|--------|-----------|
| M1：SW 显式节点映射 | R-F-001, R-F-002, R-F-011, R-F-018 | topo_mapper + pool_merge 支持 SW 节点映射 + 显式 SW 优先级 | — |
| M2：PPPoE Server 配置 | R-F-003, R-F-004, R-F-019 | switch_configurator + commands 三厂商 PPPoE 命令族 + 数据源契约 | M1 |
| M3：PPPoE Client + SW 设备 | R-F-005, R-F-009, R-F-010 | ptm-atomic op 映射 + devices.yaml switch 块 + device-reference SW 别名 | M1 |
| M4：环回 env-file 导出 | R-F-006, R-F-007, R-F-008 | exporter + case_runner env-file SW 契约 | M1 |
| M5：多实例 TRex | R-F-013, R-F-014, R-F-015, R-F-016, R-F-017 | physical_pool + pool_merge + exporter 多实例 TRex + interface_kind 透传 | M1 |
| M6：验证 | R-F-012 | trex 双向发流 0 丢包验证 | M2, M3, M4, M5 |

## 默认假设（REQUIRED 级别澄清的默认值）

| ID | 假设内容 | 关联需求 |
|----|---------|---------|
| A-038-001 | SW 进 env-file nodes 但不进 port_mapping，不新增 ${ENV.sw.*} | R-F-007, R-C-005 |
| A-038-002 | PPPoE Server 首版全局路由域，VRF 绑定可选（复用 ip binding vpn-instance） | R-F-003 |
| A-038-003 | env-file links 无向，发流方向由用例 txport/rxport 指定 | R-F-006 |

## 明确排除项（Out of Scope）

- 固化单条 PPPoE 环回链路。
- `${ENV.sw.*}` 占位符（DEF-038-01）。
- ngfw-pppoe factor-library（DEF-038-02）。
- PPPoE Server 强制 VRF 隔离（DEF-038-03）。
- SW 配置回滚闭环（DEF-038-04）。
- ptm-atomic CLI 本体修改（沿用 CR-033/037 边界）。
