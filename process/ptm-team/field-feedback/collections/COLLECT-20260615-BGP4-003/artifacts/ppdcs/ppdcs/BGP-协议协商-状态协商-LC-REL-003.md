# BGP4+ P-Parameter 判定表分析 — LC-REL-003: 链路/邻居级可靠性

> 生成时间: 2026-06-12 | 方法: parameter-design (PPDCS P-Parameter)
> design-plan.md / design-planner-reasoning.md: 缺失

## 一、设计计划与推理摘要

> **警告**: design-plan.md 和 design-planner-reasoning.md 在此仓库中均缺失，本节内容完全基于 logic-cases.md 中 LC-REL-003 的公开信息及通用 BGP4+ 协议知识推导，未经过 design-planner Skill 的正式推理流程。

**LC-REL-003 来源信息**:
- 测试逻辑：验证链路故障、keepalive异常和邻居断联后的恢复
- 判定维度: 故障类型（链路down / keepalive全部丢失 / keepalive部分丢失 / 大流量）
- Hold Timer: 180s（默认）
- 关联需求：SR-010
- 覆盖测试点: TP-Q-REL-010~013, TP-M3-003, TP-M3-005
- PPDCS 特征: P-Parameter
- CAE聚合规则: 规则1-参数化

**设计思路**: 针对 BGP4+ 链路/邻居级可靠性场景，核心目标是验证不同故障类型下 BGP 邻居状态迁移路径和恢复行为是否与协议预期一致。因子选取以"故障类型"为核心驱动因子，辅以 Hold Timer 边界参数和前/后状态因子，构建覆盖 4 种故障类型的参数组合空间。判定表使用 4 条规则，每条规则对应一种故障类型，结合前/后状态和 trace_refs 建立与逻辑用例的双向追溯。

**未满足的 design-plan 前置条件**: design-plan.md 缺失意味着本节未执行 design-planner 的 EARS→PPDCS 因子识别、topology binding 到具体组网元素、以及组合/约束消解等标准流程。实际补全时需先补齐设计计划。

## 二、Factor Catalog

| Factor ID | 名称 | 类型 | 角色 | 取值 | 说明 |
|-----------|------|------|------|------|------|
| FAC-BGP6-FAULT-TYPE | 故障类型 | enum | control, driver | {链路down, keepalive全部丢失, keepalive部分丢失, 大流量} | 核心驱动因子，决定邻居状态迁移路径 |
| FAC-BGP6-HOLD-TIMER | Hold Timer值 | data | boundary | {180s（默认）, 可配置值} | 仅 keepalive 全部丢失场景触发超时判决 |
| FAC-BGP6-PRE-STATE | 故障前邻居状态 | enum | constraint | Established | 所有测试场景的公共前置条件 |
| FAC-BGP6-RECOVERY-BEHAVIOR | 恢复后行为 | enum | observation | {自动重建→Established, 保持Established, 超时断开→Idle后重建} | 后置观测因子，反映不同故障类型的恢复差异 |

**因子选取说明**:
- `FAC-BGP6-FAULT-TYPE` 是控制因子（control/driver），决定规则分支。4 种取值对应 logic-cases.md 中判定的 4 个故障维度。
- `FAC-BGP6-HOLD-TIMER` 是边界因子（data/boundary），仅在 keepalive 丢失类故障中生效，其他故障类型中为 N/A。
- `FAC-BGP6-PRE-STATE` 是约束因子（constraint），所有场景统一为 Established，简化组合空间。
- `FAC-BGP6-RECOVERY-BEHAVIOR` 是观测因子（observation），反映恢复后邻居是否回到 Established。

## 三、Topology Binding Catalog

| 拓扑元素 | BGP 角色 | IPv6 地址 | AS 号 | 接口绑定 |
|----------|----------|-----------|-------|----------|
| DUT-A | BGP Speaker | 配置中指定 | 配置中指定 | 与 DUT-B 直连接口 |
| DUT-B | BGP Peer | 配置中指定 | 配置中指定 | 与 DUT-A 直连接口 |
| TG（Test Generator） | 流量源/控制面干扰 | N/A | N/A | 通过测试拓扑注入故障、流量 |

**说明**: design-plan.md 缺失，未完成拓扑元素到底层设备/端口的具体绑定。上表为推荐绑定结构，实际执行时需补充到具体组网图中的端口名、IP 段分配、AS 分配。

## 四、参数范围与规则清单

### 4.1 参数范围

| 参数 | 默认值 | 取值范围 | 本次取值 | 依据 |
|------|--------|----------|----------|------|
| Hold Timer | 180s | 3-65535s（RFC 4271） | 180s | logic-cases.md 默认值 |

### 4.2 业务规则（参数驱动）

| 规则 ID | 规则描述 |
|----------|----------|
| R-REL3-01 | IF 故障类型=链路down THEN 邻居状态迁移路径: Established→Active/Connect；admin up 接口后自动重建→Established |
| R-REL3-02 | IF 故障类型=keepalive全部丢失 THEN Hold Timer(180s)超时→邻居→Idle；解除阻断后重建→Established |
| R-REL3-03 | IF 故障类型=keepalive部分丢失（丢包50%）THEN Hold Timer 未触发，邻居保持 Established |
| R-REL3-04 | IF 故障类型=大流量 THEN 数据面负载不影响 BGP 控制面，邻居保持 Established |
| R-REL3-05 | IF 故障类型=keepalive全部丢失 AND 恢复时间>Hold Timer THEN 邻居状态迁移: Established→Idle→（解除阻断）→Active/Connect→Established |

## 五、判定结构（判定表）

### 5.1 条件部分

| 条件 | R1 | R2 | R3 | R4 |
|------|----|----|----|-----|
| 故障类型 (FAC-BGP6-FAULT-TYPE) | 链路down | keepalive全部丢失 | keepalive部分丢失（50%） | 大流量 |
| 故障前邻居状态 (FAC-BGP6-PRE-STATE) | Established | Established | Established | Established |
| Hold Timer (FAC-BGP6-HOLD-TIMER) | N/A | 180s | 180s | N/A |

### 5.2 动作部分

| 动作 | R1 | R2 | R3 | R4 |
|------|----|----|----|-----|
| 邻居状态迁移 (FAC-BGP6-NEIGHBOR-STATE) | Established→Active/Connect | Established→Idle（Hold超时触发） | 保持 Established | 保持 Established |
| 恢复后行为 (FAC-BGP6-RECOVERY-BEHAVIOR) | admin up→Active→OpenSent→OpenConfirm→Established | 解除阻断→Connect→OpenSent→OpenConfirm→Established | 持续保持 Established | 持续保持 Established |
| trace_refs | TP-M3-005 | TP-Q-REL-011, TP-Q-REL-013 | TP-Q-REL-012 | TP-Q-REL-010 |
| priority | P1 | P1 | P2 | P2 |
| fact_status | confirmed | confirmed | confirmed | confirmed |

### 5.3 规则行摘要

| 规则 | 故障场景 | 预期行为 | 优先级 | 关联测试点 |
|------|----------|----------|--------|-----------|
| R1 | 链路down | 邻居状态机检测到接口down，从Established回退到Active/Connect；admin up后自动重建邻居 | P1 | TP-M3-005 |
| R2 | keepalive全部丢失 | Hold Timer 180s超时后邻居转入Idle；安全策略解除后自动重建 | P1 | TP-Q-REL-011, TP-Q-REL-013 |
| R3 | keepalive部分丢失（丢包50%） | Keepalive 间隔内仍有部分报文到达，Hold Timer 不超时，邻居保持 Established | P2 | TP-Q-REL-012 |
| R4 | 大流量 | 数据面满载不影响控制面 BGP 状态机，邻居保持 Established | P2 | TP-Q-REL-010 |

## 六、Data Row → LC 叠加

以下 4 条 data row 与逻辑用例和判定规则建立双向追溯，其中每条 data row 均为完整可执行测试数据的概要描述：

### DR-REL3-01：链路 down 恢复
| 属性 | 值 |
|------|-----|
| 初始状态 | DUT-A 与 DUT-B BGP 邻居 Established |
| 注入故障 | admin shutdown DUT-A 与 DUT-B 之间的直连接口 |
| 观测中间状态 | 邻居状态: Established→Active（或 Connect，取决于实现） |
| 恢复操作 | admin up 被 shutdown 的接口 |
| 观测恢复 | 邻居状态: Active/Connect→OpenSent→OpenConfirm→Established |
| 映射规则 | R-REL3-01 |
| 映射跟踪 | TP-M3-005 |

### DR-REL3-02：keepalive 全部丢失（Hold Timer 超时）
| 属性 | 值 |
|------|-----|
| 初始状态 | DUT-A 与 DUT-B BGP 邻居 Established |
| 注入故障 | 在 DUT-A 上部署 ACL/安全策略，阻断 UDP/179 端口（或 BGP keepalive/update 报文） |
| 观测中间状态 | 180s（Hold Timer 默认值）内无 keepalive 到达 → DUT-B 邻居状态: Established→Idle |
| 观测确认 | 确认 Idle 状态持续时间 > 0，邻居不再尝试主动连接（等待 Hold Timer 超时方进入 Idle 后的 ConnectRetry 计时器行为视实现而定） |
| 恢复操作 | 移除 ACL/安全策略 |
| 观测恢复 | 邻居状态: Idle→Connect→OpenSent→OpenConfirm→Established |
| 边界验证 | 验证恢复时间 < 恢复触发后合理时间窗口（ConnectRetry 定时器 + TCP 握手 + BGP Open 交换） |
| 映射规则 | R-REL3-02, R-REL3-05 |
| 映射跟踪 | TP-Q-REL-011, TP-Q-REL-013 |

### DR-REL3-03：keepalive 部分丢失（丢包率 50%）
| 属性 | 值 |
|------|-----|
| 初始状态 | DUT-A 与 DUT-B BGP 邻居 Established |
| 注入故障 | 在 DUT-A→DUT-B 方向施加 50% 丢包（限 keepalive 报文或全报文，视工具能力） |
| 观测持续 | 保持丢包状态 ≥ 3×Hold Timer（≥540s） |
| 期望行为 | 邻居状态持续保持 Established，未触发 Hold Timer 超时（部分 keepalive 仍可达对端） |
| 映射规则 | R-REL3-03 |
| 映射跟踪 | TP-Q-REL-012 |

### DR-REL3-04：大流量数据面压力下控制面稳定性
| 属性 | 值 |
|------|-----|
| 初始状态 | DUT-A 与 DUT-B BGP 邻居 Established |
| 注入流量 | TG 持续发送线速 IPv6 数据面流量（≥90% 链路带宽） |
| 观测持续 | 保持线速流量 ≥ Hold Timer 时长（≥180s） |
| 控制面观测 | BGP 邻居状态持续保持 Established；BGP keepalive/update 报文的发送和接收不受数据面拥塞影响（控制面优先级保护） |
| 数据面观测 | 流量转发正常，无异常丢包或中断 |
| 映射规则 | R-REL3-04 |
| 映射跟踪 | TP-Q-REL-010 |

## 七、覆盖策略

### 7.1 故障类型覆盖
| 故障类型 | 规则 | Data Row | 覆盖说明 |
|----------|------|----------|----------|
| 链路down | R-REL3-01 | DR-REL3-01 | 物理/链路层故障，验证邻居状态机回退与接口恢复后自动重建 |
| keepalive全部丢失 | R-REL3-02/05 | DR-REL3-02 | 控制面消息阻断，验证 Hold Timer 超时机制和策略解除后重建 |
| keepalive部分丢失 | R-REL3-03 | DR-REL3-03 | 部分丢包场景，验证 Hold Timer 不会被非完全丢失触发 |
| 大流量 | R-REL3-04 | DR-REL3-04 | 数据面压力场景，验证控制面与数据面隔离 |

### 7.2 边界覆盖
| 边界 | 值 | 规则 | 说明 |
|------|-----|------|------|
| Hold Timer 超时边界 | 180s | R-REL3-02 | keepalive 全部丢失后，在 Hold Timer 超时前邻居应保持 Established，超时后进入 Idle |
| 部分丢包不触发边界 | 50% 丢包 | R-REL3-03 | 只要至少有 1 个 keepalive 在 Hold Timer 间隔内到达，邻居不应断开 |
| 数据面满载边界 | 线速 | R-REL3-04 | 即使数据面满载，BGP 控制面也应正常工作 |

### 7.3 测试点覆盖矩阵
| 测试点 | 规则 | Data Row | 覆盖状态 |
|--------|------|----------|----------|
| TP-M3-005 | R-REL3-01 | DR-REL3-01 | 已覆盖 |
| TP-Q-REL-010 | R-REL3-04 | DR-REL3-04 | 已覆盖 |
| TP-Q-REL-011 | R-REL3-02 | DR-REL3-02 | 已覆盖 |
| TP-Q-REL-012 | R-REL3-03 | DR-REL3-03 | 已覆盖 |
| TP-Q-REL-013 | R-REL3-02 | DR-REL3-02 | 已覆盖 |
| TP-M3-003 | — | — | 未覆盖（此测试点属于 M-分析维度，非本 P-Parameter 判定表的直接范围） |

### 7.4 未覆盖项
- TP-M3-003: 属于 M-Analyzer 维度测试点，不在 P-Parameter 判定表直接覆盖范围内，需在 M-Analyzer 阶段单独覆盖。
- Hold Timer 非默认值（可配置值）的边界测试: 当前仅覆盖默认 180s，若 SR-010 要求验证可配置 Hold Timer 在各种取值下的行为，需扩展因子取值或追加边界 Data Row。
