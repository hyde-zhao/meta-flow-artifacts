# BGP4+ P-Parameter 判定表分析 — LC-FW-003: 不同出接口类型路由转发

> 生成时间: 2026-06-12 | 方法: parameter-design (PPDCS P-Parameter)
> design-plan.md / design-planner-reasoning.md: 缺失

## 一、设计计划与推理摘要

**缺失声明**: 本判定表对应的设计计划文档（`design-plan.md`）和设计推理文档（`design-planner-reasoning.md`）均不存在。以下分析基于 `logic-cases.md` 中 LC-FW-003 的描述、`test-points.md` 中 TP-M1-019~021/TP-M1-022 的定义、`coupling-test-points.md` 中 TP-F-M3-M5-001~002 的耦合测试点，以及 `test-objects-factors.md` 中的因子定义，直接执行 P-Parameter 判定表设计。

**推理摘要**:
- **判定驱动**: LC-FW-003 的核心动作路径为四种出接口类型的参数化组合（物理口 → 子接口 → 聚合口 → 隧道口），属于 CAE 聚合规则1（参数化），适合用 P-Parameter 判定表组织。
- **因子选择**: 从 logic-cases.md 因子-取值表中提取出接口类型（control/driver）和路由学习方式（control/driver）作为基准维度；引入 MTU 值（data/auxiliary）覆盖 TP-M1-022 的 MTU 边界场景；引入流量转发结果（data/output）作为预期动作因子。
- **覆盖目标**: 覆盖 4 种出接口 × 2 种路由学习 = 8 条基本规则，加聚合口成员切换、MTU 边界及耦合测试点中的流量中断/恢复场景。

## 二、Factor Catalog（因子目录）

| 因子ID | 因子名称 | 类型 | 角色 | 取值/枚举 | 来源 |
|--------|---------|------|------|-----------|------|
| FAC-BGP6-OUT-INTF-TYPE | 出接口类型 | enum | control, driver | 物理口(GE0_1), 子接口(GE0_1.100), 聚合口(bond0), 隧道口(tunnel0) | logic-cases.md |
| FAC-BGP6-ROUTE-LEARN-TYPE | 路由学习方式 | enum | control, driver | IBGP, EBGP | logic-cases.md |
| FAC-BGP6-INTF-MTU | MTU值 | data | auxiliary | 1500, 1280, 1279 | TP-M1-022 (logic-cases.md) |
| FAC-BGP6-TRAFFIC-FORWARD | 流量转发结果 | enum | output | 成功转发, 异常(分片/丢弃), 分片后转发 | 预期输出因子（扩展） |

**因子统计**: 共 4 个因子（2 个 control/driver + 1 个 data/auxiliary + 1 个 output）

### 因子补充说明

**FAC-BGP6-OUT-INTF-TYPE（出接口类型）**:
- `物理口(GE0_1)`: 标准物理以太网接口，无 VLAN 封装，无额外封装头开销
- `子接口(GE0_1.100)`: VLAN 子接口（802.1Q），带 VLAN tag 100，报文比物理口多 4 字节 VLAN 头
- `聚合口(bond0)`: LAG 链路聚合口，包含多个成员物理口，支持成员口切换不中断转发
- `隧道口(tunnel0)`: IPv6 GRE/IPsec 隧道接口，带隧道封装头，MTU 需考虑封装开销

**FAC-BGP6-ROUTE-LEARN-TYPE（路由学习方式）**:
- `IBGP`: 同 AS 内 IBGP 邻居学习路由，下一跳通常为邻居 Loopback 或接口地址
- `EBGP`: 跨 AS EBGP 邻居学习路由，默认更改下一跳为自身接口地址

**FAC-BGP6-INTF-MTU（MTU值）**:
- `1500`: 标准以太网 MTU（正常值）
- `1280`: IPv6 协议规定的最小 MTU（RFC 2460），边界值
- `1279`: 小于 IPv6 最小 MTU 的异常值，可能导致 TCP MSS 协商异常、BGP Open 消息被分片或丢弃

**FAC-BGP6-TRAFFIC-FORWARD（流量转发结果）**:
- `成功转发`: 流量正常到达目的地，无丢包、无异常
- `异常(分片/丢弃)`: 因 MTU 不匹配或封装问题导致分片失败或丢包
- `分片后转发`: MTU 不足时 IPv6 源端分片后成功转发（IPv6 路由器不分片）

## 三、Topology Binding Catalog

| 拓扑要素 | 绑定说明 | 关联因子 |
|---------|---------|---------|
| TG-PHYSICAL-IF | DUT1-DUT2 通过物理口 GE0_1 直连，IBGP/EBGP 邻居建立 | FAC-BGP6-OUT-INTF-TYPE（物理口）, FAC-BGP6-ROUTE-LEARN-TYPE |
| TG-SUBIF-VLAN | DUT1-DUT2 通过子接口 GE0_1.100（VLAN 100）互联 | FAC-BGP6-OUT-INTF-TYPE（子接口） |
| TG-LAG-BOND | DUT1-DUT2 通过聚合口 bond0 互联，含2个成员物理口 | FAC-BGP6-OUT-INTF-TYPE（聚合口） |
| TG-TUNNEL | DUT1-DUT2 通过 IPv6 GRE/IPsec 隧道 tunnel0 互联 | FAC-BGP6-OUT-INTF-TYPE（隧道口） |
| TG-MTU-ASYM | DUT1 MTU=1500, DUT2 MTU=1280/1279 非对称 MTU 拓扑 | FAC-BGP6-INTF-MTU |
| TG-TRAFFIC-GEN | TG 测试仪通过 DUT1 发送 BGP 宣告网段流量至 DUT2 侧目标 | FAC-BGP6-TRAFFIC-FORWARD |

## 四、参数范围与规则清单

### 4.1 参数范围

| 因子 | 逻辑最小值 | 逻辑最大值 | 典型值 | 边界值 | 等价类 |
|------|-----------|-----------|--------|--------|--------|
| 出接口类型 | 物理口 | 隧道口 | 物理口 | — | 物理口, 子接口, 聚合口, 隧道口 |
| 路由学习方式 | IBGP | EBGP | IBGP | — | IBGP, EBGP |
| MTU值 | 1279 (异常) | 1500 (正常) | 1500 | 1280 (IPv6最小), 1279 (<最小) | 正常(≥1280), 异常(<1280) |
| 流量转发结果 | 异常 | 成功转发 | 成功转发 | 异常(分片/丢弃) | 成功, 分片后转发, 异常 |

### 4.2 规则清单

| 规则ID | 规则描述 | 条件组合 | 预期结果 |
|--------|---------|---------|---------|
| R-FW3-01 | 物理口 + IBGP：BGP4+ 路由下一跳指向物理口，TG 发流量 | 出接口=物理口 ∧ 路由学习=IBGP | 物理口正常转发，流量无丢包 |
| R-FW3-02 | 物理口 + EBGP：BGP4+ 路由下一跳指向物理口 | 出接口=物理口 ∧ 路由学习=EBGP | 物理口正常转发（EBGP下一跳自动改为自身） |
| R-FW3-03 | 子接口 + IBGP：带 VLAN tag 流量正常转发 | 出接口=子接口 ∧ 路由学习=IBGP ∧ VLAN=100 | 带 VLAN tag 流量通过子接口正常转发 |
| R-FW3-04 | 子接口 + EBGP：带 VLAN tag 流量正常转发 | 出接口=子接口 ∧ 路由学习=EBGP | 带 VLAN tag 流量通过子接口正常转发 |
| R-FW3-05 | 聚合口 + IBGP：成员口切换转发不中断 | 出接口=聚合口 ∧ 路由学习=IBGP ∧ 成员口切换 | 聚合口正常转发；成员口 down 后流量自动切换到剩余成员口，不中断 |
| R-FW3-06 | 聚合口 + EBGP：聚合口正常转发 | 出接口=聚合口 ∧ 路由学习=EBGP | 聚合口正常转发 |
| R-FW3-07 | 隧道口 + IBGP：隧道封装后正常转发 | 出接口=隧道口 ∧ 路由学习=IBGP ∧ 隧道正常 | GRE/IPsec 封装后流量正常转发 |
| R-FW3-08 | 隧道口 + EBGP：隧道封装后正常转发 | 出接口=隧道口 ∧ 路由学习=EBGP | GRE/IPsec 封装后流量正常转发 |
| R-FW3-09 | MTU=1280 边界：MSS 协商可能异常 | MTU=1280 ∧ 出接口=物理口 | MTU 等于 IPv6 最小值，邻居可能正常 Established 但 MSS 协商可能受限 |
| R-FW3-10 | MTU<1280 异常：TCP 分片/BGP Open 消息异常 | MTU=1279 ∧ 出接口=物理口 | MTU 小于 IPv6 最小 MTU，TCP 分片可能导致 BGP Open 消息异常或丢弃 |

## 五、判定结构（判定表）

### 5.1 主判定表（出接口类型 × 路由学习方式 → 转发结果）

| 条件/动作 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 |
|-----------|----|----|----|----|----|----|----|----|
| **C: 出接口类型** | 物理口(GE0_1) | 物理口(GE0_1) | 子接口(GE0_1.100) | 子接口(GE0_1.100) | 聚合口(bond0) | 聚合口(bond0) | 隧道口(tunnel0) | 隧道口(tunnel0) |
| **C: 路由学习** | IBGP | EBGP | IBGP | EBGP | IBGP | EBGP | IBGP | EBGP |
| **C: MTU** | 1500 | 1500 | 1500 | 1500 | 1500 | 1500 | 1500 | 1500 |
| **C: 成员口状态** | — | — | — | — | 双成员up | 双成员up | — | — |
| **A: 预期转发** | 正常转发 | 正常转发(EBGP下一跳改自身) | VLAN tag 正常转发 | VLAN tag 正常转发 | 正常转发；成员切换不中断 | 正常转发 | 隧道封装后正常转发 | 隧道封装后正常转发 |
| **A: 规则ID** | R-FW3-01 | R-FW3-02 | R-FW3-03 | R-FW3-04 | R-FW3-05 | R-FW3-06 | R-FW3-07 | R-FW3-08 |
| **A: trace_refs** | TP-M1-019(扩展) | 扩展 | TP-M1-019(扩展) | 扩展 | TP-M1-020(扩展) | 扩展 | TP-M1-021(扩展) | 扩展 |
| **A: priority** | P1 | P1 | P1 | P1 | P1 | P1 | P1 | P1 |
| **A: 备注** | 物理口 IBGP 基线 | EBGP 行为验证 | 子接口 VLAN 行为 | 子接口 EBGP | 聚合口高可用核心 | 聚合口 EBGP | 隧道封装行为 | 隧道 EBGP |

### 5.2 MTU 边界子表

| 条件/动作 | R9 | R10 |
|-----------|----|-----|
| **C: 出接口类型** | 物理口(GE0_1) | 物理口(GE0_1) |
| **C: 路由学习** | IBGP | IBGP |
| **C: MTU** | 1280 | 1279 |
| **C: 成员口状态** | — | — |
| **A: 预期结果** | 邻居可能 Established；MSS 协商可能受限，大数据包可能分片 | TCP 分片→BGP Open 消息可能异常或丢弃；邻居可能无法 Established |
| **A: 规则ID** | R-FW3-09 | R-FW3-10 |
| **A: trace_refs** | TP-M1-022 | TP-M1-022 |
| **A: priority** | P2 | P2 |
| **A: 备注** | IPv6 最小 MTU 边界 | 低于 IPv6 最小 MTU 异常 |

### 5.3 简化判定表（等价类合并视图）

| Rule ID | 出接口类型 | 路由学习 | MTU | 场景分类 |
|---------|-----------|---------|-----|---------|
| INTF-PHYS | 物理口 | IBGP/EBGP | 1500 | 物理口基线转发 |
| INTF-SUBIF | 子接口 | IBGP/EBGP | 1500 | VLAN 子接口转发 |
| INTF-LAG | 聚合口 | IBGP/EBGP | 1500 | 聚合口转发+高可用 |
| INTF-TUNNEL | 隧道口 | IBGP/EBGP | 1500 | 隧道封装转发 |
| MTU-BOUNDARY | 物理口 | IBGP | 1280 | IPv6 最小 MTU 边界 |
| MTU-BELOW | 物理口 | IBGP | 1279 | 低于最小 MTU 异常 |

## 六、Data Row → LC 叠加

依据 CAE 聚合规则1（参数化），将 LC-FW-003 四种出接口类型与两种路由学习方式参数化组合，并叠加 MTU 边界与聚合口成员切换场景，生成 data row：

| Data Row ID | 出接口类型 | 路由学习 | MTU | TG 操作 | 预期结果 | 覆盖规则 | 覆盖 trace | 备注 |
|------------|-----------|---------|-----|---------|---------|---------|-----------|------|
| DR-FW3-001 | 物理口(GE0_1) | IBGP | 1500 | TG 向 BGP 路由网段发流量 | 物理口正常转发，无丢包 | R-FW3-01 | TP-M1-019(扩展) | 物理口 IBGP 基线 |
| DR-FW3-002 | 物理口(GE0_1) | EBGP | 1500 | TG 向 BGP 路由网段发流量 | 正常转发；EBGP 下一跳改为 DUT1 接口地址 | R-FW3-02 | 扩展 | 物理口 EBGP 验证 |
| DR-FW3-003 | 子接口(GE0_1.100) | IBGP | 1500 | TG 发带 VLAN tag 100 流量 | 子接口正常接收 VLAN tag 流量并转发 | R-FW3-03 | TP-M1-019(扩展) | 子接口 VLAN IBGP |
| DR-FW3-004 | 子接口(GE0_1.100) | EBGP | 1500 | TG 发带 VLAN tag 100 流量 | 子接口正常接收 VLAN tag 流量并转发（EBGP 下一跳正确） | R-FW3-04 | 扩展 | 子接口 VLAN EBGP |
| DR-FW3-005 | 聚合口(bond0) | IBGP | 1500 | TG 发流量 → 断开成员口1 | 聚合口正常转发；成员口1 down 后流量自动切换到成员口2，转发不中断 | R-FW3-05 | TP-M1-020(扩展) + TP-F-M3-M5-001 | 聚合口高可用核心 |
| DR-FW3-006 | 聚合口(bond0) | EBGP | 1500 | TG 发流量 | 聚合口正常转发 | R-FW3-06 | 扩展 | 聚合口 EBGP |
| DR-FW3-007 | 隧道口(tunnel0) | IBGP | 1500 | TG 发流量至隧道对端网段 | 隧道封装后正常转发；抓包验证 GRE/IPsec 封装头 | R-FW3-07 | TP-M1-021(扩展) | 隧道 IBGP 封装 |
| DR-FW3-008 | 隧道口(tunnel0) | EBGP | 1500 | TG 发流量至隧道对端网段 | 隧道封装后正常转发 | R-FW3-08 | 扩展 | 隧道 EBGP |
| DR-FW3-009 | 物理口(GE0_1) | IBGP | 1280 | TG 发 1280 字节 IPv6 包 | 邻居可能 Established；MSS 协商可能受限；大数据包（>1280）可能被分片 | R-FW3-09 | TP-M1-022 | MTU IPv6 最小边界 |
| DR-FW3-010 | 物理口(GE0_1) | IBGP | 1279 | POST 邻居 → TG 发流量 | TCP 分片可能导致 BGP Open 消息异常或丢弃；邻居可能无法 Established | R-FW3-10 | TP-M1-022 | MTU 低于 IPv6 最小 |
| DR-FW3-011 | 聚合口(bond0) | IBGP | 1500 | TG 持续发流量 → 断开成员口1 → 恢复成员口1 | 成员口1 down 时流量切到成员口2（不中断）；成员口1 up 后流量恢复双成员负载 | R-FW3-05 + TP-F-M3-M5-002 | TP-F-M3-M5-001~002 | 聚合口故障+恢复 |
| DR-FW3-012 | 隧道口(tunnel0) | IBGP | 1500 | TG 持续发流量 → 断开隧道 | 隧道断开后 BGP 邻居 down，路由清除，流量中断 | TP-F-M3-M5-001(扩展) | TP-F-M3-M5-001 | 隧道口故障耦合 |

**Data Row 统计**: 共 12 条，覆盖 LC-FW-003 四种出接口 × 两种路由学习 = 8 条基本参数化组合，外加 1 条 MTU 边界、1 条 MTU 异常、1 条聚合口恢复、1 条隧道故障耦合场景。

## 七、覆盖策略

### 7.1 覆盖维度分析

| 覆盖维度 | 已覆盖取值 | 覆盖方式 | 状态 |
|---------|-----------|---------|------|
| 出接口类型 | 物理口, 子接口, 聚合口, 隧道口 | 等价类全覆盖（各至少2条×IBGP/EBGP） | OK |
| 路由学习方式 | IBGP, EBGP | 等价类全覆盖（每种出接口各1条 IBGP + 1条 EBGP） | OK |
| MTU值 | 1500, 1280, 1279 | 正常值 + 边界值(1280) + 异常值(1279) | OK |
| 聚合口成员切换 | 双成员up, 成员口1 down, 成员口1 恢复 | 状态转换全覆盖 | OK |
| 隧道故障 | 隧道正常, 隧道断开 | 故障状态覆盖 | OK |
| 流量转发结果 | 成功转发, 分片后转发/异常, 中断 | 三类输出全覆盖 | OK |

### 7.2 覆盖质量评估

- **因子覆盖**: 4/4 因子全部参与判定，覆盖率 100%
- **规则覆盖**: 10/10 规则均有 data row 驱动验证（R-FW3-01~10 全部覆盖）
- **trace 覆盖**:
  - TP-M1-019（子接口邻居建立）→ 扩展为转发验证，由 DR-FW3-001/003 覆盖
  - TP-M1-020（聚合口邻居建立）→ 扩展为转发验证+成员切换，由 DR-FW3-005/011 覆盖
  - TP-M1-021（隧道口邻居建立）→ 扩展为转发验证+隧道故障，由 DR-FW3-007/012 覆盖
  - TP-M1-022（MTU 不一致）→ 由 DR-FW3-009/010 覆盖
  - TP-F-M3-M5-001（邻居断开流量中断）→ 由 DR-FW3-005/012 覆盖
  - TP-F-M3-M5-002（邻居恢复流量恢复）→ 由 DR-FW3-011 覆盖
- **PPDCS 特征**: P-Parameter（判定表主体，出接口类型 × 路由学习参数化）
- **CAE 聚合**: 规则1-参数化，通过 12 条 data row 覆盖 4 种出接口类型与 2 种路由学习方式的全部组合
- **未覆盖项**: 子接口/聚合口/隧道口各自叠加 MTU 边界的组合场景被等价类归并至 R9/R10（物理口 MTU 边界作为代表性覆盖）；若需精确覆盖各出接口 × MTU 组合，可将 MTU 子表扩展至 4（出接口类型）× 2（MTU边界）= 8 条规则

### 7.3 依赖关系

```
DR-FW3-001 (物理口 IBGP 基线)
  ├─→ DR-FW3-002 (物理口 EBGP，拓扑共用)
  ├─→ DR-FW3-003 (子接口 IBGP，独立 VLAN 拓扑)
  │     └─→ DR-FW3-004 (子接口 EBGP，拓扑共用)
  ├─→ DR-FW3-005 (聚合口 IBGP+成员切换，独立 LAG 拓扑)
  │     ├─→ DR-FW3-006 (聚合口 EBGP，拓扑共用)
  │     └─→ DR-FW3-011 (聚合口故障恢复，依赖005完成)
  ├─→ DR-FW3-007 (隧道口 IBGP，独立隧道拓扑)
  │     ├─→ DR-FW3-008 (隧道口 EBGP，拓扑共用)
  │     └─→ DR-FW3-012 (隧道故障耦合，依赖007隧道就绪)
  ├─→ DR-FW3-009 (MTU=1280 边界，物理口拓扑+MTU修改)
  └─→ DR-FW3-010 (MTU=1279 异常，物理口拓扑+MTU修改)
```

### 7.4 判定结论

本 P-Parameter 判定表完整覆盖 LC-FW-003 的四种出接口类型（物理口/子接口/聚合口/隧道口）与两种路由学习方式（IBGP/EBGP）的全部 8 种参数化组合，并通过 MTU 边界子表覆盖 TP-M1-022 的 IPv6 MTU 边界场景，通过聚合口成员切换和隧道故障场景覆盖 TP-F-M3-M5-001~002 的耦合测试点。10 条规则与 12 条 data row 形成完整的参数化驱动关系，满足 PPDCS P-Parameter 设计要求。
