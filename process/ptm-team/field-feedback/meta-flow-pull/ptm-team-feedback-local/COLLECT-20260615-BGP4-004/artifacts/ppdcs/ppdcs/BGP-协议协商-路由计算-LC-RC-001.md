# BGP4+ P-Parameter 判定表分析 — LC-RC-001: 路由选路

> 生成时间: 2026-06-12 | 设计 Skill: parameter-design
> 上游输入: logic-cases.md §LC-RC-001, test-objects-factors.md, test-points.md
> PPDCS 主特征: P-Parameter
> CAE 聚合规则: 规则1-参数化
> 本文件覆盖 LC-RC-001（BGP4+ 路由选路）的 P-Parameter 判定表分析，包括主判定表（路径来源 × 对比来源 → 选路结果）和故障切换子表。

---

## 一、设计计划与推理摘要

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | **缺省** — 设计计划文件不存在，以用户直接指定的 factor 定义 + logic-cases.md + test-points.md 为设计输入 |
| design-planner-reasoning.md | **缺省** — 设计推理记录文件不存在，推理过程未归档；本节注明此缺口 |
| logic-cases.md | LC-RC-001 测试逻辑：验证 MED 最低优先、EBGP 优于 IBGP、本地路由优先等选路规则；动作路径：建立多邻居 → 不同来源宣告相同路由 → 验证 BGP4+ 路由表按优先级选择 |
| test-objects-factors.md | FAC-BGP-ROUTE-SELECT: 路由选路规则, enum(MED最低,EBGP优先,本地优先), control, driver, P-Parameter |
| test-points.md | TP-M4-001~004, TP-F-M2-M4-001~002 |

### 设计缺口声明

> **注意**: `design-plan.md` 和 `design-planner-reasoning.md` 均不存在。本次 P-Parameter 分析以 `logic-cases.md` 中 LC-RC-001 的定义、`test-objects-factors.md` 中的关联因子、以及 `test-points.md` 中的覆盖测试点作为直接设计输入。因子目录、拓扑绑定、参数范围与判定表均由本分析自主推导，未经过 design-planner 阶段的形式化推理与备案。后续若补充 design-plan，建议对比本分析与 formal plan 的一致性。

### 判定维度与动作路径对齐

| 维度 | 内容 |
|------|------|
| 判定维度（logic-cases.md） | 路径来源(IBGP MED=100 / IBGP MED=50 / EBGP / 本地) → 选路结果(MED低优先 / EBGP优 / 本地优) |
| 动作路径（logic-cases.md） | 建立多邻居 → 不同来源宣告相同路由 → 验证 BGP4+ 路由表按优先级选择 |
| 因子系统映射 | 路径来源 ↦ FAC-BGP6-PATH-SOURCE, MED 值 ↦ FAC-BGP6-MED-VALUE, 选路链 ↦ FAC-BGP6-ROUTE-SELECT-RULE |

---

## 二、Factor Catalog（因子目录）

| factor_id | 因子名称 | 值域摘要 | 参数类型(control/data) | 角色(driver/constraint) | PPDCS 特征 | 绑定拓扑节点 | source_refs | fact_status | 备注 |
|-----------|---------|---------|----------------------|------------------------|------------|-------------|-------------|:---:|------|
| FAC-BGP6-PATH-SOURCE | 路径来源类型 | enum(IBGP_MED100, IBGP_MED50, EBGP, LOCAL) | control | driver | P-Parameter | DUT_A, DUT_C, DUT_D | LC-RC-001, TP-M4-001~004 | confirmed | 含 MED 值锚定的 IBGP 子型 |
| FAC-BGP6-ROUTE-SELECT-RULE | 选路规则优先级链 | enum(MED最低→EBGP优先→本地优先) | control | driver | P-Parameter | DUT_B | LC-RC-001, FAC-BGP-ROUTE-SELECT | confirmed | 选路决策逻辑的抽象因子 |
| FAC-BGP6-MED-VALUE | MED 对比值 | 50 vs 100, 0(边界) | data | driver | P-Parameter | DUT_A, DUT_C | TP-M4-001, TP-M4-002 | confirmed | MED 值数据因子，驱动 MED 比较规则 |
| FAC-BGP6-LOCAL-ROUTE-PRESENT | 本地路由存在性 | enum(存在, 不存在) | control | driver | P-Parameter | DUT_B | TP-M4-003 | confirmed | 决定本地优先规则是否触发 |
| FAC-BGP6-NEIGHBOR-STATE | 邻居状态 | enum(正常, 断开) | data | driver | P-Parameter | DUT_C(EBGP) | TP-M4-004 | confirmed | 故障注入因子 |

---

## 三、Topology Binding Catalog（拓扑绑定目录）

### 3.1 拓扑节点映射

| 节点标识 | 逻辑角色 | 拓扑绑定描述 | 关联因子 |
|---------|---------|-------------|---------|
| DUT_A | IBGP 邻居（MED=100 / MED=50） | DUT_B 的 IBGP 对等体，宣告目标路由，MED 值可配置 | FAC-BGP6-PATH-SOURCE(IBGP_MED100 / IBGP_MED50), FAC-BGP6-MED-VALUE |
| DUT_B | 选路设备（被测设备） | 执行 BGP4+ 选路决策的设备，拥有指向 A、C、D 的多条路径 | FAC-BGP6-ROUTE-SELECT-RULE, FAC-BGP6-LOCAL-ROUTE-PRESENT |
| DUT_C | EBGP 邻居（或替代 IBGP MED=50） | DUT_B 的另一对等体，宣告与 DUT_A 相同的路由，邻居类型可切换 | FAC-BGP6-PATH-SOURCE(EBGP / IBGP_MED50), FAC-BGP6-NEIGHBOR-STATE |
| DUT_D | 本地路由设备 | 与 DUT_B 共享同一网段的设备（或 DUT_B 自身配置本地接口地址） | FAC-BGP6-PATH-SOURCE(LOCAL) |

### 3.2 拓扑拓扑图（逻辑）

```
        ┌──────────┐
        │  DUT_A   │  IBGP 邻居
        │ MED=100  │──┐
        │(或 MED=50)│  │  IBGP Session
        └──────────┘  │
                      ▼
                ┌──────────┐       ┌──────────┐
                │  DUT_B   │◄─────►│  DUT_C   │
                │ (选路设备) │ EBGP  │ EBGP/IBGP│
                │          │ 或IBGP│ MED=50   │
                └──────────┘       └──────────┘
                      ▲
                      │ 本地直连
                ┌──────────┐
                │  DUT_D   │
                │ (同网段)  │
                └──────────┘
```

### 3.3 绑定约束

| 约束编号 | 约束描述 | 适用规则 |
|---------|---------|---------|
| BIND-01 | DUT_A 与 DUT_C 宣告的 NLRI（目标路由前缀）必须相同 | 规则1~5, 7 |
| BIND-02 | DUT_B 本地路由前缀（来自 DUT_D 直连接口）与 A/C 宣告的 NLRI 相同时，触发本地优先 | 规则1 |
| BIND-03 | DUT_A 与 DUT_C 不能同时对 DUT_B 使用 EBGP（否则削弱 IBGP vs EBGP 对比） | 规则2, 4 |
| BIND-04 | 故障切换子表中，仅 DUT_C 的 EBGP 邻居断开；DUT_A 的 IBGP 邻居保持 | 规则4 |

---

## 四、参数范围与规则清单

### 4.1 选路优先级链（完整覆盖）

BGP4+ 选路决策按以下优先级链执行，高优先级规则先匹配，匹配成功后不再评估后续规则：

```
本地路由 (最高优先)
  └─► EBGP 路径
      └─► IBGP 路径（含 MED 比较，取最小值）
          └─► 单路径（唯一可用路径，兜底）
```

### 4.2 规则清单

| 规则编号 | 规则名称 | 规则描述 | 触发条件 | 预期结果 | 覆盖测试点 |
|---------|---------|---------|---------|---------|-----------|
| 规则1 | 本地优先 | 当 DUT_B 存在同网段本地接口路由时，优先选本地路由，不安装 BGP 路由 | DUT_B 本地路由存在 且 NLRI 匹配 | 路由表选中本地接口，BGP 路由不安装 | TP-M4-003 |
| 规则2 | EBGP 优于 IBGP | 同一 NLRI，EBGP 路径优先于 IBGP 路径，不论 MED 值如何 | 存在 EBGP 和 IBGP 两条路径 | 选 EBGP 路径 | TP-M4-002 |
| 规则3 | MED 较低优先 | 同属 IBGP（或同属 EBGP）的多条路径，比较 MED 值，取 MED 最小者 | 多条同类型路径，MED 值不同 | 选 MED 值较小的路径 | TP-M4-001 |
| 规则4 | EBGP 失效切换 IBGP | 当前活跃路径为 EBGP，EBGP 邻居断开时，切换到次优 IBGP 路径 | EBGP 邻居断开，IBGP 路径存在 | 路由切换到 IBGP 路径 | TP-M4-004 |
| 规则5 | IBGP MED=50 vs MED=100 | 两条 IBGP 路径分别宣告 MED=50 和 MED=100，验证选 MED=50 | DUT_A(MED=100) + DUT_C(MED=50) 同时宣告 | 选 DUT_C(MED=50) | TP-M4-001 |
| 规则6 | 单路径兜底 | 当某 NLRI 仅有一条可用路径（无论 EBGP 还是 IBGP），直接选中 | 仅一条 BGP 路径有效 | 选该唯一路径 | — |
| 规则7 | MED=0 边界 | MED=0 是协议定义的最小合法值，验证极端低 MED 优先 | 一条路径 MED=0，另一条 MED=50/100 | 选 MED=0 路径 | — |

---

## 五、判定结构（判定表）

### 5.1 主判定表：路径来源 → 选路结果

| 条件 / 动作 | Rule-01 | Rule-02 | Rule-03 | Rule-04 | Rule-05 |
|:-----------|:-------:|:-------:|:-------:|:-------:|:-------:|
| **条件** | | | | | |
| DUT_B 到 A 的路径类型 | IBGP | IBGP | IBGP | EBGP | 本地 |
| DUT_B 到 C 的路径类型 | IBGP | EBGP | 无 | 无 | IBGP |
| DUT_A 的 MED 值 | 100 | — | — | — | — |
| DUT_C 的 MED 值 | 50 | — | — | — | — |
| DUT_B 本地路由（同网段） | 无 | 无 | 无 | 无 | 有 |
| EBGP 邻居状态 | 正常 | 正常 | — | — | — |
| **动作** | | | | | |
| 选路结果 | 选 C（MED 低） | 选 C（EBGP 优） | 选 A（唯一路径） | 选 C（EBGP 单路径） | 本地接口（本地优先） |
| 命中规则 | 规则3/5: MED 低优先 | 规则2: EBGP > IBGP | 规则6: 单路径兜底 | 规则2: EBGP > 无 | 规则1: 本地优先 |
| 路由表条目 | C 的 BGP 路由 installed | C 的 EBGP 路由 installed | A 的 IBGP 路由 installed | C 的 EBGP 路由 installed | A/C 的 BGP 路由 not installed，本地路由 active |
| **覆盖测试点** | TP-M4-001 | TP-M4-002 | — | — | TP-M4-003 |

### 5.2 故障切换子表：邻居状态变化 → 选路迁移

| 条件 / 动作 | Rule-06 | Rule-07 |
|:-----------|:-------:|:-------:|
| **条件** | | |
| 初始活跃路径 | EBGP（C 侧） | IBGP（A 侧） |
| 切换前选路结果 | 选 C（EBGP） | 选 A（IBGP） |
| 故障注入 | DUT_C EBGP 邻居断开 | 无故障（基线） |
| DUT_A IBGP 邻居状态 | 正常 | 正常 |
| DUT_C 替代路径 | IBGP（A 侧）可用 | — |
| **动作** | | |
| 切换后选路结果 | 切换到 IBGP（A 侧） | 保持 IBGP（A 侧） |
| 命中规则 | 规则4: EBGP 失效 → 切换 IBGP | 规则6: 单路径保持 |
| 收敛预期 | BGP 路由收敛，流量路径迁移 | 无变化 |
| **覆盖测试点** | TP-M4-004 | — |

### 5.3 MED=0 边界判定表（扩展）

| 条件 / 动作 | Rule-08 |
|:-----------|:-------:|
| **条件** | |
| DUT_B 到 A 的路径类型 | IBGP |
| DUT_B 到 C 的路径类型 | IBGP |
| DUT_A 的 MED 值 | 100 |
| DUT_C 的 MED 值 | 0 |
| DUT_B 本地路由 | 无 |
| **动作** | |
| 选路结果 | 选 C（MED=0，最小边界值） |
| 命中规则 | 规则7: MED=0 优先于 MED=100 |
| 边界属性 | MED 值域下界验证 |
| **覆盖测试点** | TP-F-M2-M4-001（MED 边界值） |

---

## 六、Data Row → LC 叠加

以下 data row 将判定表规则映射到具体测试数据，确保每条规则至少有一条 data row 覆盖。

| Data-Row | LC 映射 | 场景描述 | A 路径类型 | A MED | C 路径类型 | C MED | 本地路由 | 邻居状态 | 预期选路结果 | 命中规则 | 覆盖测试点 |
|:--------:|:------:|---------|:--------:|:-----:|:--------:|:-----:|:------:|:------:|-------------|:------:|-----------|
| DR-01 | LC-RC-001 | IBGP MED=100 vs IBGP MED=50 | IBGP | 100 | IBGP | 50 | 无 | 正常 | 选 C（MED=50，MED 低优先） | 规则3/5 | TP-M4-001 |
| DR-02 | LC-RC-001 | IBGP vs EBGP（相同路由） | IBGP | — | EBGP | — | 无 | 正常 | 选 C（EBGP 优先） | 规则2 | TP-M4-002 |
| DR-03 | LC-RC-001 | 本地路由 vs BGP 路由 | IBGP | — | IBGP | — | 有（同网段） | 正常 | 本地接口（本地优先，BGP 路由不安装） | 规则1 | TP-M4-003 |
| DR-04 | LC-RC-001 | EBGP 断开 → 切换到 IBGP | IBGP | 100 | EBGP | — | 无 | EBGP 断开 | 初始选 EBGP → 断开后选 IBGP(A) | 规则4 | TP-M4-004 |
| DR-05 | LC-RC-001 | 仅 IBGP 单路径 | IBGP | 100 | 无 | — | 无 | 正常 | 选 A（唯一 IBGP 路径） | 规则6 | — |
| DR-06 | LC-RC-001 | 仅 EBGP 单路径 | 无 | — | EBGP | — | 无 | 正常 | 选 C（唯一 EBGP 路径） | 规则6 | — |
| DR-07 | LC-RC-001 | MED=0 边界验证 | IBGP | 100 | IBGP | 0 | 无 | 正常 | 选 C（MED=0 最小，MED 低优先） | 规则7 | TP-F-M2-M4-001 |

### 6.1 Data Row 与判定表 Rule 的覆盖矩阵

| | Rule-01 | Rule-02 | Rule-03 | Rule-04 | Rule-05 | Rule-06 | Rule-07 | Rule-08 |
|:-:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
| DR-01 | | | ✓ | | | | | |
| DR-02 | | ✓ | | | | | | |
| DR-03 | ✓ | | | | | | | |
| DR-04 | | | | ✓ | | | | |
| DR-05 | | | | | | ✓ | | |
| DR-06 | | | | | | ✓ | | |
| DR-07 | | | | | | | ✓ | |
| DR-08 | | | | | | | | ✓ |

注：DR-05 和 DR-06 分别覆盖 IBGP 单路径和 EBGP 单路径两种 Rule-06 场景；DR-07 和 DR-08 分别覆盖 MED=0 和 MED 比较两个 MED 维度场景。

### 6.2 Data Row 扩展条目（MED 对比细化）

| Data-Row | A 路径类型 | A MED | C 路径类型 | C MED | 预期选路结果 | 说明 |
|:--------:|:--------:|:-----:|:--------:|:-----:|-------------|------|
| DR-08 | IBGP | 100 | IBGP | 50 | 选 C | MED 值较小者优先，DR-01 等价 |
| DR-09 | IBGP | 50 | IBGP | 100 | 选 A | MED 反转验证，确认 MED 较低规则非偶然 |
| DR-10 | IBGP | 0 | IBGP | 100 | 选 A | MED=0 最小边界，DR-07 等价 |

---

## 七、覆盖策略

### 7.1 选路规则覆盖矩阵

| 选路规则 | 是否覆盖 | 覆盖方式 | 关联 Data Row | 关联测试点 |
|---------|:------:|---------|:-----------:|----------|
| MED 比较（低优先） | ✓ | 判定表 Rule-03/05，Data Row DR-01, DR-07~DR-10 | DR-01, DR-07 | TP-M4-001, TP-F-M2-M4-001 |
| EBGP 优于 IBGP | ✓ | 判定表 Rule-02/04，Data Row DR-02 | DR-02 | TP-M4-002 |
| 本地路由优先 | ✓ | 判定表 Rule-01，Data Row DR-03 | DR-03 | TP-M4-003 |
| 故障回退（EBGP→IBGP） | ✓ | 判定表 Rule-06，Data Row DR-04 | DR-04 | TP-M4-004 |
| 单路径兜底（IBGP） | ✓ | 判定表 Rule-06，Data Row DR-05 | DR-05 | — |
| 单路径兜底（EBGP） | ✓ | 判定表 Rule-06，Data Row DR-06 | DR-06 | — |

### 7.2 优先级链完整覆盖验证

选路优先级链 `本地 > EBGP > IBGP（MED 低优先）` 的三层结构逐层覆盖：

| 优先级层级 | 覆盖判定表 Rule | Data Row | 描述 |
|:--------:|:-------------:|:--------:|------|
| L1: 本地优先 | Rule-01 | DR-03 | 本地路由高于一切 BGP 路由 |
| L2: EBGP 优先 | Rule-02, Rule-04 | DR-02, DR-04, DR-06 | EBGP 优先于 IBGP；EBGP 失效时下落到 L3 |
| L3: IBGP MED 低优先 | Rule-03/05, Rule-07 | DR-01, DR-05, DR-07~DR-10 | IBGP 内部按 MED 最小值选路 |

### 7.3 边界覆盖

| 边界类型 | 边界值 | 覆盖条目 | 描述 |
|---------|-------|:------:|------|
| MED 下限 | MED = 0 | DR-07 | BGP4+ 协议定义的最小合法 MED 值，验证 MED=0 路径被优先选中 |
| MED 反转 | MED(A)=50, MED(C)=100 | DR-09 | 与 DR-01 的 MED 赋值互换，证实选路结果由 MED 值决定，非节点固定偏好 |
| 无竞争 | 仅一条 BGP 路径 | DR-05, DR-06 | 单路径场景，确保无 MED/EBGP 竞争时不会误选或路由丢失 |
| 本地抢占 | 本地路由 vs 所有 BGP 路由 | DR-03 | 本地路由最高优先级，确保 BGP 路由被正确抑制 |

### 7.4 测试点覆盖汇总

| 测试点 | 覆盖状态 | 覆盖 Data Row | 验证目标 |
|-------|:------:|:------------:|---------|
| TP-M4-001 | ✓ | DR-01 | A(MED=100)+C(MED=50) IBGP → 选 C(MED 低) |
| TP-M4-002 | ✓ | DR-02 | A(IBGP)+C(EBGP) 相同路由 → 选 C(EBGP) |
| TP-M4-003 | ✓ | DR-03 | DUT2 宣告 /48，DUT1 配本地 /48 → 本地优先 |
| TP-M4-004 | ✓ | DR-04 | EBGP 路径在用，断 EBGP 邻居 → 切 IBGP |
| TP-F-M2-M4-001 | ✓ | DR-07 | MED=0 边界值覆盖 |
| TP-F-M2-M4-002 | ✓ | DR-02（组合覆盖） | EBGP vs IBGP + MED 对比组合场景 |

### 7.5 未覆盖项与遗留风险

| 风险项 | 描述 | 影响 | 建议 |
|-------|------|------|------|
| Local Preference 未纳入 | BGP4+ 选路决策中 Local Preference 优先级高于 MED，本分析聚焦 MED 维度，未纳入 Local Preference 因子 | local-pref 为默认值场景等价，非默认值时规则序列变化 | 若后续扩展 Local Preference 测试，需追加 P-Parameter 因子 FAC-BGP6-LOCAL-PREF 并在判定表中插入更高优先级 Rule |
| AS-Path 长度未纳入 | 在 MED 比较之前，AS-Path 长度是比较条件，本分析假设所有路径 AS-Path 长度相同 | 同 AS 场景默认满足 | 跨 AS 场景需追加 AS-Path 因子 |
| 多路径负载均衡 | BGP4+ 支持 ECMP，本分析聚焦单最优路径选路 | 多路径同时活跃的场景未覆盖 | 如需 ECMP 测试，追加 P-Parameter 因子 FAC-BGP6-MULTIPATH |

---

## 八、参考资料

| 参考文件 | 用途 | 可用状态 |
|---------|------|:------:|
| logic-cases.md §LC-RC-001 | 测试逻辑、判定维度、动作路径定义 | ✓ |
| test-objects-factors.md | FAC-BGP-ROUTE-SELECT, FAC-BGP6-MED-COMPARE 因子定义 | ✓ |
| test-points.md | TP-M4-001~004, TP-F-M2-M4-001~002 覆盖测试点 | ✓ |
| ppdcs-annotation.md | PPDCS 特征标注（P-Parameter 主特征确认） | ✓ |
| design-plan.md | 设计计划（**缺省**） | ✗ |
| design-planner-reasoning.md | 设计推理记录（**缺省**） | ✗ |
| RFC 4271 / RFC 2545 | BGP4 / BGP4+ 选路标准 | 外部参考 |
