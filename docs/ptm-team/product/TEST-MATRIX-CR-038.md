# CR-038 测试覆盖矩阵（scenario-expansion 产出，meta-qa 消费）

> 覆盖对象：CR-038 PPPoE 链路规划能力（SW 节点映射 + PPPoE Server/Client 配置 + 环回拓扑 + 验证）。
> 覆盖状态：`covered` / `n/a` / `waived` / `uncovered`（需说明风险）。

| 场景 ID | 类别 | 需求/来源 | 覆盖状态 | 验证方式 | 风险/备注 |
|---|---|---|---|---|---|
| SCN-038-POS-001 | 正向 | R-F-001 | covered | static/dry-run（topo_mapper 单测） | 无 |
| SCN-038-POS-002 | 正向 | R-F-002 | covered | static/dry-run（topo_mapper 单测） | 无 |
| SCN-038-POS-003 | 正向 | R-F-003 | covered | dry-run（switch_configurator 生成 diff） | 需 H3C 真机核对命令，独立授权 |
| SCN-038-POS-004 | 正向 | R-F-004 | covered | static（commands 模板渲染单测）+ dry-run diff | H3C 真机验证；ruijie/huawei 仅 dry-run diff 供人工核对（未真机验证） |
| SCN-038-POS-005 | 正向 | R-F-005 | covered | runtime（NGFW web，独立授权） | ptm-atomic op 可用性待 LLD 核实（RA-038-001） |
| SCN-038-POS-006 | 正向 | R-F-006 | covered | static/dry-run（exporter 单测） | 无 |
| SCN-038-POS-007 | 正向 | R-F-007 | covered | static/dry-run（exporter 单测） | 无 |
| SCN-038-POS-008 | 正向 | R-F-009/010 | covered | static（devices.yaml/device-reference 校验） | 无 |
| SCN-038-POS-009 | 正向 | R-F-012 | covered | runtime（trex 发流，独立授权） | 需真机授权 |
| SCN-038-POS-010 | 正向 | R-F-013/014/015 | covered | static/dry-run（physical_pool + pool_merge + exporter 单测） | 多实例字段消费，向后兼容 node2 系列需回归 |
| SCN-038-POS-011 | 正向 | R-F-016/017 | covered | static/dry-run（exporter 单测） | interface_kind=pppoe 不产 ip + next_hop null |
| SCN-038-NEG-001 | 负向 | R-NF-002 | covered | static/dry-run（映射失败单测） | 无 |
| SCN-038-NEG-002 | 负向 | R-NF-002 | covered | static/dry-run（映射失败单测） | 无 |
| SCN-038-NEG-003 | 负向 | R-C-003 | covered | dry-run（授权阻断单测） | 无 |
| SCN-038-NEG-004 | 负向 | R-C-001 | covered | static（产物凭据扫描） | 无 |
| SCN-038-NEG-005 | 负向 | R-F-006 | covered | static/dry-run（exporter 回归） | 无 |
| SCN-038-BND-001 | 边界 | R-NF-004 | covered | static/dry-run（CR-037 回归） | 无 |
| SCN-038-BND-002 | 边界 | R-F-001 | covered | static/dry-run（Mock 单测） | 无 |
| SCN-038-BND-003 | 边界 | R-C-006 | covered | static（IPAM 校验） | 依赖 SGA-01 结论（待 CP2） |
| SCN-038-BND-004 | 边界 | R-F-006 | covered | static/dry-run（exporter links 校验单测） | 环回 links 去重/自环检测/闭环完整性校验 |
| SCN-038-AUTH-001 | 权限 | R-C-003 | covered | review（runtime_authorization 决策项） | 无 |
| SCN-038-AUTH-002 | 权限 | R-C-004 | covered | review（改动范围检查） | 无 |
| SCN-038-REC-001 | 失败恢复 | R-NF-003 | covered | dry-run（台账占用单测） | 无 |
| SCN-038-REC-002 | 失败恢复 | DEF-038-04 | waived | — | DEF-038-04 延后，CP7 重启 |
| SCN-038-PRE-001 | precheck | R-F-005 | covered | review（LLD 预检） | 跨仓库依赖风险 |
| SCN-038-PRE-002 | precheck | R-F-001 | covered | review（调研确认） | physical_pool SW 已预留 |

## 覆盖缺口摘要

| 状态 | 数量 | 场景 | 风险 |
|---|---|---|---|
| uncovered | 0 | — | 无 |
| n/a | 0 | — | 无 |
| waived | 1 | SCN-038-REC-002 | SW 配置回滚闭环延后至 CP7（DEF-038-04） |

## 风险提示

- SCN-038-POS-003 / POS-005 / POS-009 依赖真机（H3C / NGFW / trex），均需独立 runtime_authorization，CP 批准不隐含真机授权。
- RA-038-001（ptm-atomic PPPoE op 可用性）在 LLD 阶段必须闭环，否则 SCN-038-POS-005 无法落地；缺失时按 DQ-038-03 三选一处理（见交还摘要待决策清单）。
- SGA-01（PPPoE 动态地址 vs 静态 IPAM）结论待 CP2，影响 SCN-038-BND-003 的 IPAM 集成实现。
- SCN-038-POS-010（多实例 TRex）需回归 node2 系列单实例向后兼容（R-F-015 回退逻辑）。
- SCN-038-BND-004（环回完整性）为新增边界场景，覆盖审核意见 6（links 去重/自环/闭环）。
