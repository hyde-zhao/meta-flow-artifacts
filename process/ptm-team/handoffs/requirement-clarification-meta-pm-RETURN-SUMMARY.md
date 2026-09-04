# 交还摘要 — requirement-clarification（meta-pm）→ host-orchestrator

- CR：CR-038（PPPoE 链路规划能力）
- 阶段：requirement-clarification
- 委托：host-orchestrator 委托 meta-pm 直连用户
- 交还时间：2026-08-15
- 下一动作：host-orchestrator 汇总并发起 CP2 正式人工确认（meta-pm 不自行发起）

---

## 1. 用户真实意图

1. 在 CR-037「逻辑 topo → 物理 topo 映射」基础上，ptm-team 扩展 **PPPoE 链路规划能力**（通用 topo 引擎能力，非固化单条链路），兑现 CR-037 HLD §1.3「后续 CR 扩展」承诺。
2. 支持 **SW 节点显式纳入自动拓扑映射**（突破单 TG+单 DUT MVP 边界），并覆盖 **PPPoE Server（SW 侧）/ PPPoE Client（NGFW 侧）配置 + 环回拓扑 env-file + SW 设备 schema/别名 + 双向发流 0 丢包验证**。
3. 支持 **多实例 TRex**（TG1 主机并跑 GE :8000 / TE :8001 两实例），topo-config/topo-planning 消费 `trex_instances` 汇总块 + 接口级 `instance` 归属字段（CP2 反馈范围补充）。
4. 严格守住安全约束：凭据 `${ENV_VAR}` 占位、local-user cipher 密文、默认 dry-run、真机下发独立 runtime_authorization、GE1_1~4 禁改动。

## 2. Scenario Gray Areas 处理结果

| 灰区 ID | 主题 | 用户选择 | 处理状态 | 讨论日志 / checkpoint |
|---|---|---|---|---|
| SGA-01 | PPPoE 地址池动态 vs 静态 IPAM | 待 CP2（SGQ-038-01） | open | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md` |
| SGA-02 | SW 参与映射后 IP/VLAN 归属 | 待 CP2（SGQ-038-01） | open | 同上 |
| SGA-03 | 环回链路方向性 | 待 CP2（SGQ-038-02） | open | 同上 |
| SGA-04 | PPPoE Server 是否强制 VRF | 待 CP2（SGQ-038-03） | open | 同上 |

- 讨论日志：`process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md`
- 恢复点：`process/checks/CP2-DISCUSSION-CHECKPOINT.json`
- SGQ 证据：SGQ-038-01（SGA-01+02，最高价值）、SGQ-038-02（SGA-03）、SGQ-038-03（SGA-04）三条用户可见场景确认交互已记录（含候选选项、推荐项、影响面），用户回答待 CP2 回填。

## 3. Deferred Ideas

| ID | 内容 | 重启条件 |
|---|---|---|
| DEF-038-01 | `${ENV.sw.*}` 占位符 | 未来用例需 case_steps 引用 SW 端口 |
| DEF-038-02 | ngfw-pppoe factor-library | 未来需 PPPoE 测试设计因子 |
| DEF-038-03 | PPPoE Server 强制 VRF | 未来拨号需 VRF 隔离 |
| DEF-038-04 | SW 配置回滚闭环 | CP7 暴露回滚需求 |

## 4. 需求摘要

- `REQUIREMENTS-CR-038.md`：功能需求 R-F-001~019（P0 12 条、P1 7 条）、约束 R-C-001~006、非功能 R-NF-001~004、风险 RA-038-001~007、里程碑 M1~M6。
- BLOCKING 未决项：0（所有歧义已通过 DQ/SGQ 记录推荐方案，待 CP2 用户确认）。
- `ready_for_design`：**false**（待 CP2 用户确认后置 true）。

## 5. SCENARIOS 覆盖摘要

`SCENARIOS-CR-038.yaml`：正向 11、负向 5、边界 4、权限 2、失败恢复 2、precheck 2，共 26 条。

## 6. TEST-MATRIX 缺口摘要

`TEST-MATRIX-CR-038.md`：covered 25、waived 1（SCN-038-REC-002，SW 配置回滚闭环 DEF-038-04 延后）。uncovered 0、n/a 0。
风险：SCN-038-POS-003/005/009 依赖真机（独立授权）；RA-038-001（ptm-atomic op 可用性）按 DQ-038-03 三选一闭环；新增 POS-010/011 多实例 TRex + BND-004 环回完整性校验。

## 7. STORY-MAP 摘要

`STORY-MAP-CR-038.md`：13 Story（P0 7、P1 6）+ 5 Epic。来源 UC-038-01~06 / REQ 已标注。新增 STORY-038-13 多实例 TRex（P0）。

## 8. MVP 范围

`MVP-SCOPE-CR-038.md`：
- **In**：SW 显式映射（含显式 SW 优先级）、PPPoE Server/Client 配置、PPPoE 数据源契约、环回 env-file、多实例 TRex、SW schema/别名、双向发流验证、安全约束。
- **Out**：固化单链路、`${ENV.sw.*}`、ngfw-pppoe factor-library、强制 VRF、ptm-atomic 本体修改。
- **Deferred**：DEF-038-01~04。
- 推荐理由：以「参考场景 6 设备环回链路可跑通」为最小验收口径，保持引擎通用性（节点类型判定而非固化链路）。

## 9. 发布切片与 backlog

- `RELEASE-SLICES-CR-038.md`：6 切片（M1 SW 映射 → M2 PPPoE Server → M3 PPPoE Client+SW 设备 → M4 环回 env-file → M5 多实例 TRex → M6 验证集成）。
- `BACKLOG-CR-038.md`：后续候选 DEF-038-01~04；阻塞前置 RA-038-001、DQ-038-05、SGA-01、SGA-04；风险 RA-038-002/004/007。

## 10. CP1 / CP2 证据

- `process/checks/CP1-CR038.md`：结论 **PASS**。
- `process/checks/CP2-CR038.md`：结论 **PASS**（未豁免 FAIL 0 项，待 CP2 人工确认决策项）。
- 讨论恢复点：`process/checks/CP2-DISCUSSION-CHECKPOINT.json`。

## 11. 待人工决策项（供 CP2 Decision Brief）

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣 | 影响/风险 | 回退/切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-038-01 | scope | PPPoE Server 建模 | 逻辑 pppoe 节点改真实 SW（node_type=SW, role=pppoe-server） | 保留 Mock + 映射期替换 | 推荐：复用 SW 匹配/role 字段、最简、无需特殊路径；备选：需新增 Mock→SW 替换逻辑，复杂度高 | 影响 topo_mapper/建模文档；不改则 Mock 无法下发配置 | 若发现 SW 节点建模破坏 Mock 语义则回退 |
| DQ-038-02 | architecture | SW env-file 契约 | SW 进 nodes，不进 port_mapping，不新增 ${ENV.sw.*} | (B) SW 全量进 env-file；(C) SW 独立产物 | 推荐：复用 CR-037 部署链路、case-execution 零适配；B 破坏 9 类契约；C 多一个产物面 | 影响 exporter/case_runner/ADR-09；B 需改 case-execution 占位符 | 未来用例需直接操作 SW 时切换 B |
| DQ-038-03 | scope | PPPoE Client 归属 + op 缺失处理路径 | ptm-atomic 原子操作；op 缺失三选一（本仓库 op_mapper 映射到可用 op / 提外部依赖 / 降级手工步骤） | factor-library 因子扩展（设计层） | 推荐：拨号是运行时配置动作，非测试设计变量；op 缺失首选本仓库 op_mapper 映射（最简） | 影响 op_mapper/因子库边界；op 缺失不闭环则 SCN-038-POS-005 无法落地 | op 缺失且无可用映射时提外部依赖或降级手工步骤 |
| SGQ-038-01 | scope | PPPoE 地址池 vs 静态 IPAM + SW IP/VLAN 归属 | 方案 A（SW 配置与地址池解耦，动态地址不进静态 IPAM） | B（SW 全量进 env-file）/ C（SW 独立产物） | 推荐 A：最简、复用 CR-037 链路、PPPoE 动态地址语义天然属配置参数 | 影响 IPAM 集成（R-C-006）；错误会破坏 env-file 静态 IP 规划语义 | 若需用例静态引用拨号地址则切 B |
| SGQ-038-02 | implementation | 环回 links 方向性 | links 无向，发流方向由用例 txport/rxport 指定 | links 有向 | 推荐：与 CR-037 links 无向端点对契约一致，方向本就用例层关注点 | 影响 exporter links 契约；加 direction 字段属过度设计 | 若后续需表达单向链路再增 direction |
| SGQ-038-03 | architecture | PPPoE Server 是否强制 VRF | 首版全局路由域，VRF 可选（复用 ip binding vpn-instance） | 首版强制 VRF | 推荐：参考场景未涉及 VRF，最小实现；备选：无谓复杂度 | 影响 PPPoE 命令族；强制 VRF 需额外 vpn-instance 配置 | 未来拨号需 VRF 隔离时切换 |
| DQ-038-05 | architecture | PPPoE 地址池网段 + local-user 数据源契约 | physical_pool 新字段（与设备台账同源，与 NGFW 拨号 163.0.0.x 同源一致） | devices.yaml / 独立配置文件 | 推荐：physical_pool 已是设备台账真相源，ip pool 段与设备强绑定；devices.yaml 与 SW 设备分离，独立配置引入第三产物面 | 影响 R-F-019 数据读取路径；不同源会破坏"同源一致"约束 | 若 ip pool 段需跨设备共享则改独立配置 |

**归 agent 默认处理（不进待决策清单）**：
- 显式 SW 节点优先级高于自动透传（R-F-018）：用户已在审核意见 1 明确方向（显式优先，已显式映射 SW 不参与透传），互斥逻辑细节进 HLD 由 meta-se 定义，不新增 DQ。

## 12. N/A / WAIVED 项

| 项 | 原因 | 影响范围 | 后续触发条件 |
|---|---|---|---|
| `${ENV.sw.*}` 占位符 | 当前用例不直接操作 SW（DEF-038-01） | case-execution 契约不变 | 未来用例需操作 SW 端口 |
| SW 配置回滚闭环（SCN-038-REC-002） | 首版只做下发+台账占用（DEF-038-04） | teardown 撤销命令未覆盖 | CP7 暴露回滚需求 |
| ptm-atomic CLI 本体修改 | 沿用 CR-033/037 边界 | 不触碰 ptm-atomic 本体 | 若 PPPoE op 缺失需跨仓库新增 |
| 既有 `docs/product/USE-CASES.md` / `REQUIREMENTS.md` 覆盖 | CR-038 是新 CR，无既有基线可追溯，采用 `-CR-038` 后缀新增文档 | 不破坏旧基线 | 未来 CR 需增量更新时追加修订记录 |

## 13. CP2 Decision Brief 输入（建议 host-orchestrator 使用）

- **推荐决策**：approve（接受 7 项推荐方案：DQ-038-01/02/03/05 + SGQ-038-01/02/03）。
- **不授权项**：approve 不隐含真机下发授权（H3C telnet / NGFW web / trex 发流均为独立 runtime_authorization）；不授权改动 GE1_1~4 实例；不授权凭据明文落盘。
- **approve 后发生**：host-orchestrator 回填 CP2 人工结果，消费 CR-038 route_plan 连续推进后续无门禁阶段，直至 CP3（HLD 门，meta-se 委托）。
- **不确认阻塞**：CP2 未通过前不得进入 solution-design / Story 拆解 / LLD / 实现。
- **成功指标**：SM-038-01~04（SW 映射 100%、PPPoE 命令 H3C 真机验证 + ruijie/huawei dry-run diff、环回 env-file 可导出、0 丢包）。
- **风险**：RA-038-001（ptm-atomic op 可用性，DQ-038-03 三选一闭环）、RA-038-002（H3C 命令厂商差异）、RA-038-003（IPAM 语义冲突）、RA-038-004（环回组合爆炸）、RA-038-007（真机下发 × 无 teardown 撤销，风险接受）。

---

## 产物路径清单

- `docs/product/USE-CASES-CR-038.md`
- `docs/product/REQUIREMENTS-CR-038.md`
- `docs/product/SCENARIOS-CR-038.yaml`
- `docs/product/TEST-MATRIX-CR-038.md`
- `docs/product/STORY-MAP-CR-038.md`
- `docs/product/MVP-SCOPE-CR-038.md`
- `docs/product/RELEASE-SLICES-CR-038.md`
- `docs/product/BACKLOG-CR-038.md`
- `process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md`
- `process/checks/CP2-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP1-CR038.md`
- `process/checks/CP2-CR038.md`
- `process/CLARIFICATION-LOG.md`（追加调研发现段落）
