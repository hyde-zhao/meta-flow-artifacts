# MVP 范围 — CR-038（PPPoE 链路规划能力）

## In（MVP 范围）

| 项 | 说明 | 理由 |
|---|---|---|
| SW 显式节点映射 | topo_mapper 支持 `node_type=SW` 显式匹配（显式 SW 优先级高于自动透传），`role=pppoe-server` 识别 | 用户拍板边界（SW 纳入自动映射）+ 审核意见 1 |
| PPPoE Server 配置 | switch_configurator + commands 三厂商 PPPoE 命令族（H3C 参考） | CR-038 核心能力 |
| PPPoE 数据源契约 | ip pool 网段 + local-user 账号数据源唯一（见 DQ-038-05） | 审核意见 2 地基 |
| PPPoE Client 拨号 | ptm-atomic 原子操作（执行层）+ op_mapper 映射 | CR-038 核心能力 |
| 环回拓扑 env-file | exporter 突破 MVP 单 TG+单 DUT，支持 TG+DUT+SW 环回（含 links 去重/自环/闭环校验） | 参考场景要求 + 审核意见 6 |
| 多实例 TRex | physical_pool + pool_merge + exporter 消费/归并/导出多实例（trex_instance/trex_sync_port/trex_async_port + interface_kind 透传） | 参考场景 TG1 并跑 GE/TE 两实例 |
| SW 设备 schema/别名 | devices.yaml sw 块 + device-reference SW 别名 | SW 寻址下发前置 |
| PPPoE 链路验证 | trex 双向发流 0 丢包 | 参考场景验收 |
| 安全约束 | ${ENV_VAR} 占位、cipher 密文、dry-run 默认、GE 实例禁改动 | 安全硬约束 |

## Out of Scope（明确排除）

- 固化单条 PPPoE 环回链路（交付形态是通用引擎能力）。
- `${ENV.sw.*}` 占位符（SW 不进 port_mapping）。
- ngfw-pppoe factor-library（设计层因子）。
- PPPoE Server 强制 VRF 隔离。
- ptm-atomic CLI 本体修改。

## Deferred（延后）

| 项 | 延后原因 | 重启条件 |
|---|---|---|
| `${ENV.sw.*}` 占位符（DEF-038-01） | 当前用例不直接操作 SW | 未来用例需 case_steps 引用 SW 端口 |
| ngfw-pppoe factor-library（DEF-038-02） | PPPoE 拨号是执行动作 | 未来需 PPPoE 测试设计因子 |
| PPPoE Server 强制 VRF（DEF-038-03） | 参考场景未涉及 VRF | 未来拨号需 VRF 隔离 |
| SW 配置回滚闭环（DEF-038-04） | 首版只做下发 + 台账占用 | CP7 暴露回滚需求 |

## 推荐理由

MVP 范围以「参考场景 6 设备环回链路可跑通」为最小验收口径，同时保持 topo 引擎通用性（SW 节点类型判定而非固化链路）。安全约束全部纳入 MVP（不可延后），真机下发独立 runtime_authorization。

## 决策项（待 CP2）

| 决策 ID | 类型 | 问题 | 推荐 | 备选 |
|---|---|---|---|---|
| DQ-038-01 | scope | PPPoE Server 建模（Mock→SW） | 改为真实 SW 节点 | 保留 Mock+映射期替换 |
| DQ-038-02 | architecture | SW env-file 契约 | SW 进 nodes 不进 port_mapping | SW 全量进 env-file / SW 独立产物 |
| DQ-038-03 | scope | PPPoE Client 归属 + op 缺失处理路径 | ptm-atomic 原子操作；op 缺失三选一（本仓库 op_mapper 映射到可用 op / 提外部依赖 / 降级手工步骤） | factor-library 扩展 |
| SGQ-038-01 | scope | PPPoE 地址池 vs 静态 IPAM + SW IP/VLAN 归属 | 方案 A（SW 配置与地址池解耦） | B（SW 全量进 env-file）/ C（SW 独立产物） |
| SGQ-038-02 | implementation | 环回 links 方向性 | links 无向，方向由用例指定 | links 有向 |
| SGQ-038-03 | architecture | PPPoE Server 是否强制 VRF | 首版全局路由域，VRF 可选 | 首版强制 VRF |
| DQ-038-05 | architecture | PPPoE 地址池网段 + local-user 数据源契约 | physical_pool 新字段（与设备台账同源） | devices.yaml / 独立配置文件 |
