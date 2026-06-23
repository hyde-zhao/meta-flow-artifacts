# BGP4+ P-Parameter 判定表分析 — LC-CMP-001: 异构兼容与路由优先级

> 生成时间: 2026-06-12 | 方法: parameter-design (PPDCS P-Parameter)
> design-plan.md / design-planner-reasoning.md: 缺失

## 一、设计计划与推理摘要

因 design-plan.md 与 design-planner-reasoning.md 均缺失，本节依据 logic-cases.md 中 LC-CMP-001 的定义进行自包含设计推理。

**测试逻辑主旨**：验证 BGP4+ 与华为/华三/锐捷三家主流国产厂商 IPv6 BGP 的异构互通能力，以及与策略路由、静态路由、OSPFv3 之间的路由优先级行为。

**判定架构**：采用双层主判定表：
- **子表 A（异构互通）**：以「厂商 × MD5 密码一致性」为条件维度，判定邻居建立与路由学习结果。
- **子表 B（路由优先级）**：以「BGP4+ vs 目标路由协议」为条件维度，结合 AD 值判定最终选路结果。

**动作路径**：P1（异构互通）→ P2（优先级对比），覆盖 8 条核心规则与 11 条以上 data row。

## 二、Factor Catalog

共 5 个因子：

| 序号 | 因子 ID | 中文名 | 取值枚举 | 角色 | 备注 |
|------|---------|--------|----------|------|------|
| 1 | FAC-BGP6-VENDOR | 异构厂商 | 华为, 华三, 锐捷 | control, driver | 决定互通测试的厂商侧条件 |
| 2 | FAC-BGP6-CMP-MD5 | MD5 互通状态 | 密码一致, 密码不一致 | control, driver | 决定邻居能否建立的关键因子 |
| 3 | FAC-BGP6-PRIORITY-COMPARE | 路由优先级对比类型 | BGPvs策略路由, BGPvs静态路由, BGPvsOSPFv3 | control, driver | 决定优先级对比的协议组合 |
| 4 | FAC-BGP6-ROUTE-AD | 管理距离 AD 值 | 策略路由AD, 静态=1, OSPFv3=110, EBGP=20, IBGP=200 | constraint | 用于解释选路结果的参考值，非直接操纵因子 |
| 5 | FAC-BGP6-CMP-RESULT | 互通结果/选路结果 | Established/失败, 某协议优先 | outcome | 观测型结果因子 |

## 三、Topology Binding Catalog

| 拓扑 ID | 拓扑描述 | 关联 DUT 角色 | 对端设备 | 适用规则 |
|---------|----------|---------------|----------|----------|
| TOPO-CMP-DUAL | DUT ↔ 单异构对端，双栈 BGPv4+BGP4+ | DUT 运行 BGP4+ | 华为/华三/锐捷各一 | R-CMP-01~05 |
| TOPO-CMP-MULTI-PROTO | DUT 同时运行 BGP4+、策略路由、静态路由、OSPFv3，指向同一目的前缀 | DUT 多协议共存 | 对端路由器 | R-CMP-06~08 |

## 四、参数范围与规则清单

### 4.1 异构互通规则

| 规则 ID | 规则内容 | 触发条件 | 预期结果 |
|---------|----------|----------|----------|
| R-CMP-01 | IF 厂商=华为 AND MD5=密码一致 THEN 邻居 Established, 双向路由学习正常 | VENDOR=华为, MD5=一致 | Established, 路由双向同步 |
| R-CMP-02 | IF 厂商=华三 AND MD5=密码一致 THEN 邻居 Established, 双向路由学习正常 | VENDOR=华三, MD5=一致 | Established, 路由双向同步 |
| R-CMP-03 | IF 厂商=锐捷 AND MD5=密码一致 THEN 邻居 Established, 双向路由学习正常 | VENDOR=锐捷, MD5=一致 | Established, 路由双向同步 |
| R-CMP-04 | IF 厂商=华为 AND MD5=密码不一致 THEN 邻居无法建立，日志报 MD5 认证失败 | VENDOR=华为, MD5=不一致 | 邻居 Idle/Connect, MD5 mismatch log |
| R-CMP-05 | BGPv4(IPv4) 与 BGP4+(IPv6) 同时运行，地址族独立，互不干扰 | BGPv4 & BGP4+ co-deployed | 两地址族邻居各自独立建立/维护 |

### 4.2 路由优先级规则

| 规则 ID | 规则内容 | 触发条件 | 预期结果 |
|---------|----------|----------|----------|
| R-CMP-06 | IF 目的前缀相同 AND 策略路由(PBR)已配置 THEN 策略路由优先于 BGP4+ | PBR 匹配 → 下一跳覆盖 BGP 路由 | PBR 下一跳生效，BGP 路由被旁路 |
| R-CMP-07 | IF 目的前缀相同 AND 静态路由(AD=1) AND IBGP(AD=200) THEN 静态路由优先加入路由表 | 静态 AD < IBGP AD | 静态路由 active，IBGP 路由为 backup |
| R-CMP-08 | IF 目的前缀相同 AND OSPFv3(AD=110) AND IBGP(AD=200) THEN OSPFv3 优先加入路由表 | OSPFv3 AD < IBGP AD | OSPFv3 路由 active，IBGP 路由为 backup |
| R-CMP-09 | IF 目的前缀相同 AND OSPFv3(AD=110) AND EBGP(AD=20) THEN EBGP 优先加入路由表 | EBGP AD < OSPFv3 AD | EBGP 路由 active，OSPFv3 路由为 backup |
| R-CMP-10 | IF 目的前缀相同 AND 静态路由(AD=1) AND EBGP(AD=20) THEN 静态路由优先加入路由表 | 静态 AD < EBGP AD | 静态路由 active，EBGP 路由为 backup |

## 五、判定结构

### 5.1 主判定表 A — 异构互通

| 条件/动作 | R1 | R2 | R3 | R4 | R5 |
|-----------|----|----|----|----|----|
| FAC-BGP6-VENDOR | 华为 | 华三 | 锐捷 | 华为 | —（任意） |
| FAC-BGP6-CMP-MD5 | 密码一致 | 密码一致 | 密码一致 | 密码不一致 | — |
| 邻居建立 | Established | Established | Established | 失败（Idle/Connect） | BGPv4 与 BGP4+ 各自 Established |
| 路由学习 | 双向正常 | 双向正常 | 双向正常 | 无法学习 | BGPv4/BGP4+ 各自独立，互不干扰 |
| 适用规则 | R-CMP-01 | R-CMP-02 | R-CMP-03 | R-CMP-04 | R-CMP-05 |
| trace_refs | TP-Q-CMP-001, TP-Q-CMP-004 | TP-Q-CMP-002 | TP-Q-CMP-003 | TP-Q-CMP-004 扩展 | TP-Q-CMP-005 |

### 5.2 主判定表 B — 路由优先级

| 条件/动作 | R6 | R7 | R8 |
|-----------|----|----|----|
| FAC-BGP6-PRIORITY-COMPARE | BGPvs策略路由 | BGPvs静态路由 | BGPvsOSPFv3 |
| 对比协议 AD 值 | PBR（策略路由覆盖） | 静态 AD=1 < IBGP AD=200 | OSPFv3 AD=110 < IBGP AD=200 |
| 选路结果 | 策略路由优先 | 静态路由优先 | OSPFv3 优先 |
| 适用规则 | R-CMP-06 | R-CMP-07 | R-CMP-08 |
| trace_refs | TP-F-M2-PBR-001 | TP-F-M2-STATIC-PRI-001 | TP-F-M2-OSPFv3-PRI-001 |

### 5.3 扩展判定表 B' — EBGP 参与优先级对比（补充）

| 条件/动作 | R9 | R10 |
|-----------|----|----|
| FAC-BGP6-PRIORITY-COMPARE | BGPvsOSPFv3 | BGPvs静态路由 |
| BGP 类型 | EBGP (AD=20) | EBGP (AD=20) |
| 对比协议 AD 值 | EBGP AD=20 < OSPFv3 AD=110 | 静态 AD=1 < EBGP AD=20 |
| 选路结果 | EBGP 优先 | 静态路由优先 |
| 适用规则 | R-CMP-09 | R-CMP-10 |
| trace_refs | TP-F-M2-OSPFv3-PRI-001 补充 | TP-F-M2-STATIC-PRI-001 补充 |

## 六、Data Row → LC 叠加

依据判定表展开 data row：

| Row ID | VENDOR | MD5 | PRIORITY-COMPARE | BGP 类型 | 预期结果 | 覆盖规则 | 关联 trace_ref |
|--------|--------|-----|------------------|----------|----------|----------|----------------|
| DR-CMP-001 | 华为 | 一致 | — | — | Established, 双向正常 | R-CMP-01 | TP-Q-CMP-001, TP-Q-CMP-004 |
| DR-CMP-002 | 华三 | 一致 | — | — | Established, 双向正常 | R-CMP-02 | TP-Q-CMP-002 |
| DR-CMP-003 | 锐捷 | 一致 | — | — | Established, 双向正常 | R-CMP-03 | TP-Q-CMP-003 |
| DR-CMP-004 | 华为 | 不一致 | — | — | 邻居建立失败 | R-CMP-04 | TP-Q-CMP-004 扩展 |
| DR-CMP-005 | — | — | — | — | BGPv4/BGP4+ 互不干扰 | R-CMP-05 | TP-Q-CMP-005 |
| DR-CMP-006 | — | — | BGPvs策略路由 | — | PBR 优先 | R-CMP-06 | TP-F-M2-PBR-001 |
| DR-CMP-007 | — | — | BGPvs静态路由 | IBGP | 静态路由优先 | R-CMP-07 | TP-F-M2-STATIC-PRI-001 |
| DR-CMP-008 | — | — | BGPvsOSPFv3 | IBGP | OSPFv3 优先 | R-CMP-08 | TP-F-M2-OSPFv3-PRI-001 |
| DR-CMP-009 | — | — | BGPvsOSPFv3 | EBGP | EBGP 优先 | R-CMP-09 | TP-F-M2-OSPFv3-PRI-001 补充 |
| DR-CMP-010 | — | — | BGPvs静态路由 | EBGP | 静态路由优先 | R-CMP-10 | TP-F-M2-STATIC-PRI-001 补充 |
| DR-CMP-011 | 锐捷 | 不一致 | — | — | 邻居建立失败 | R-CMP-04 扩展 | — |

> 共计 11 条 data row，覆盖 3 厂商 × 2 MD5 状态 + 1 双栈共存 + 3 优先级对比 + 2 EBGP 补充 + 1 扩展异常。

## 七、覆盖策略

| 覆盖维度 | 覆盖项 | 对应规则 | 覆盖判定 |
|----------|--------|----------|----------|
| 异构厂商 | 华为 | R-CMP-01, R-CMP-04 | 已覆盖 |
| 异构厂商 | 华三 | R-CMP-02 | 已覆盖 |
| 异构厂商 | 锐捷 | R-CMP-03 | 已覆盖 |
| MD5 密码一致性 | 密码一致 | R-CMP-01~03 | 已覆盖（3 厂商） |
| MD5 密码一致性 | 密码不一致 | R-CMP-04, DR-CMP-011 | 已覆盖（华为 + 锐捷） |
| 双栈共存 | BGPv4 + BGP4+ 互不干扰 | R-CMP-05 | 已覆盖 |
| 路由优先级对比 | PBR 优先 | R-CMP-06 | 已覆盖 |
| 路由优先级对比 | 静态 > IBGP | R-CMP-07 | 已覆盖 |
| 路由优先级对比 | OSPFv3 > IBGP | R-CMP-08 | 已覆盖 |
| 路由优先级对比 | EBGP > OSPFv3 | R-CMP-09 | 已覆盖 |
| 路由优先级对比 | 静态 > EBGP | R-CMP-10 | 已覆盖 |

**覆盖结论**：11 条 data row 覆盖 LC-CMP-001 定义的全部 5 个 trace_ref 测试点（TP-Q-CMP-001~005, TP-F-M2-PBR-001, TP-F-M2-STATIC-PRI-001, TP-F-M2-OSPFv3-PRI-001），并扩展了 EBGP 场景与锐捷 MD5 不一致场景，确保异构兼容+路由优先级两个子领域的判定覆盖完备。
