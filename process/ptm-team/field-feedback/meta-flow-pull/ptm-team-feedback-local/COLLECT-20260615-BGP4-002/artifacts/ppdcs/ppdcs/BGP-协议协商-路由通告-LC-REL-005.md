# BGP4+ P-Parameter 判定表分析 — LC-REL-005: 容量/性能边界

> 生成时间: 2026-06-12 | 方法: parameter-design (PPDCS P-Parameter)
> 已确认取值: 邻居上限=16, ECMP最大路径=16, 收敛≤5s, 建立≤5s
> design-plan.md / design-planner-reasoning.md: 缺失

## 一、设计计划与推理摘要

> **注意**: `design-plan.md` 和 `design-planner-reasoning.md` 文件不存在于当前特性工作区中。本分析基于 `logic-cases.md` 中 LC-REL-005 的定义和 `test-objects-factors.md` 中已确认的取值直接构建。

LC-REL-005 属于 P-Parameter（参数化）判定类型，核心目标是验证 BGP4+ 在规格上限和容量边界下的行为。判定逻辑围绕五个因子的组合展开：邻居数量边界、路由表容量、收敛时间达标性、邻居建立时间达标性，以及组合压力模式。

四个独立维度（邻居规格、路由容量、收敛时间、建立时间）各自由 2~3 个取值构成边界判定表；此外通过组合压力模式将多因子叠加，验证极限场景下的系统稳定性。规则总数 11 条，覆盖 7 个性能测试点 (TP-Q-PERF-001~007)。

## 二、Factor Catalog

| 因子 ID | 因子名称 | 取值 | 取值说明 | 来源 |
|---------|----------|------|----------|------|
| FAC-BGP6-NEIGHBOR-COUNT | 邻居数量 | 16 / 17 | 16=规格内(上限), 17=超规格(越界) | test-objects-factors.md |
| FAC-BGP6-ROUTE-CAPACITY | BGP4+ 路由表容量 | 5K / 50K / 160K | 随设备类型不同: 5K=全系列, 50K=高端, 160K=旗舰 | test-objects-factors.md |
| FAC-BGP6-CONVERGE-TIME | 路由收敛时间 | ≤5s / >5s | ≤5s=达标, >5s=不达标 | test-objects-factors.md |
| FAC-BGP6-ESTABLISH-TIME | 邻居建立时间 | ≤5s / >5s | POST 到 Established 耗时, ≤5s=达标, >5s=不达标 | test-objects-factors.md |
| FAC-BGP6-STRESS-MODE | 压力模式 | 仅邻居规格 / 仅路由容量 / 邻居+路由组合 / 收敛+建立 | 控制压力维度组合方式 | logic-cases.md |

## 三、Topology Binding Catalog

| 拓扑角色 | 设备标识 | 数量 | 角色说明 |
|----------|----------|------|----------|
| DUT1 | 被测设备 | 1 | BGP4+ 能力验证目标 |
| DUT2~DUT17 | 邻居设备 | 16 | 模拟 BGP4+ 邻居，覆盖规格上限 |
| TG | 流量仪/路由注入器 | 1 | 注入 BGP4+ 路由、发送线速流量、测量时间 |

拓扑连接：TG 连接 DUT1；DUT1 分别与 DUT2~DUT17 建立 BGP4+ 邻居关系。所有邻居运行双栈 (IPv4+IPv6) 或纯 IPv6 BGP4+。

## 四、参数范围与规则清单

### 规则定义

| 规则 ID | 规则条件 | 规则结论 | 关联因子 | 优先级 |
|---------|----------|----------|----------|--------|
| R-REL5-01 | IF 邻居数 ≤ 16 THEN | 全部进入 Established 状态，内存和 CPU 占用在可接受范围内 | FAC-BGP6-NEIGHBOR-COUNT | P1 |
| R-REL5-02 | IF 邻居数 = 17 (越界) THEN | POST 返回错误，第 17 个邻居创建被拒绝 | FAC-BGP6-NEIGHBOR-COUNT | P1 |
| R-REL5-03 | IF 路由容量 = 5K THEN | 全部路由学习成功，无内存溢出 | FAC-BGP6-ROUTE-CAPACITY | P1 |
| R-REL5-04 | IF 路由容量 = 50K THEN | 全部路由学习成功，内存增长 < 20% | FAC-BGP6-ROUTE-CAPACITY | P1 |
| R-REL5-05 | IF 路由容量 = 160K THEN | 全部路由学习成功，无内存溢出（旗舰设备） | FAC-BGP6-ROUTE-CAPACITY | P1 |
| R-REL5-06 | IF 收敛时间 ≤ 5s THEN | 达标 | FAC-BGP6-CONVERGE-TIME | P1 |
| R-REL5-07 | IF 收敛时间 > 5s THEN | 不达标 [待确认: 实际测量方法与阈值依据] | FAC-BGP6-CONVERGE-TIME | P2 |
| R-REL5-08 | IF 建立时间 ≤ 5s THEN | 达标 | FAC-BGP6-ESTABLISH-TIME | P1 |
| R-REL5-09 | IF 建立时间 > 5s THEN | 不达标 | FAC-BGP6-ESTABLISH-TIME | P2 |
| R-REL5-10 | IF 16 邻居 Established + 160K 路由 + TG 线速流量 THEN | 邻居全部保持、路由不丢失、CPU/内存不超限 | FAC-BGP6-NEIGHBOR-COUNT × FAC-BGP6-ROUTE-CAPACITY × FAC-BGP6-STRESS-MODE | P1 |
| R-REL5-11 | IF 16 邻居每个宣告 100 条路由 THEN | 路由表 1600 条正确，持续 1h，内存 CPU 稳定 | FAC-BGP6-NEIGHBOR-COUNT × FAC-BGP6-ROUTE-CAPACITY × FAC-BGP6-STRESS-MODE | P2 |

### CAE 聚合规则

采用 **规则1-参数化** 聚合：因子取值组合形成判定规则，按等价类划分覆盖规格内/超规格边界。每个因子的每个有效取值至少出现在一条规则中；组合规则 (R10, R11) 覆盖多因子交互的极限场景。

## 五、判定结构

### 5.1 邻居规格判定表

| 条件/动作 | R1 (规格内) | R2 (越界) |
|-----------|-------------|-----------|
| FAC-BGP6-NEIGHBOR-COUNT | 16 | 17 |
| 创建操作 | POST bgp6neighbor × 16 | POST bgp6neighbor × 16 + POST 第 17 个 |
| 预期结果 | 全部 Established | 前 16 个 Established；第 17 个返回错误 |
| trace_refs | TP-Q-PERF-001 | TP-Q-PERF-002 |
| priority | P1 | P1 |

### 5.2 路由容量判定表

| 条件/动作 | R3 (5K) | R4 (50K) | R5 (160K) |
|-----------|---------|----------|-----------|
| FAC-BGP6-ROUTE-CAPACITY | 5K | 50K | 160K |
| 适用设备类型 | 全系列 | 高端设备 | 旗舰设备 |
| 学习结果 | 全部成功 | 全部成功 | 全部成功 |
| 资源约束 | 无内存溢出 | 内存增长 < 20% | 无内存溢出 |
| trace_refs | TP-Q-PERF-005 | TP-Q-PERF-006 | TP-Q-PERF-007 |
| priority | P1 | P1 | P1 |

### 5.3 收敛/建立时间判定表

| 条件/动作 | R6 (收敛达标) | R7 (收敛不达标) | R8 (建立达标) | R9 (建立不达标) |
|-----------|---------------|-----------------|---------------|-----------------|
| 时间类型 | 收敛时间 | 收敛时间 | 建立时间 | 建立时间 |
| 时间值 | ≤5s | >5s | ≤5s | >5s |
| 判定 | 达标 | 不达标 | 达标 | 不达标 |
| trace_refs | TP-Q-PERF-003 | TP-Q-PERF-003 (扩展) | TP-Q-PERF-004 | TP-Q-PERF-004 (扩展) |
| priority | P1 | P2 | P1 | P2 |

### 5.4 组合压力判定表

| 条件/动作 | R10 (极限组合) | R11 (中等组合长稳) |
|-----------|----------------|---------------------|
| FAC-BGP6-NEIGHBOR-COUNT | 16 | 16 |
| 路由数 | 160K | 100 × 16 = 1600 |
| 流量 | TG 线速流量 | — |
| 持续时间 | 持续运行 | 1h |
| 预期结果 | 邻居保持 + 路由不丢 + CPU/内存不超限 | 路由表正确 + 内存 CPU 稳定 |
| trace_refs | 组合压力 (P8) | 组合压力 (P5) |
| priority | P1 | P2 |

## 六、Data Row → LC 叠加

基于上述判定表，推导出至少 11 条数据行 (data row)，每条对应一个测试场景：

| 序号 | 数据行描述 | 对应规则 | 关键参数 | 预期结果 | 关联测试点 |
|------|-----------|----------|----------|----------|------------|
| 1 | 创建 16 个 BGP4+ 邻居 | R-REL5-01 | NEIGHBOR-COUNT=16 | 全部 Established | TP-Q-PERF-001 |
| 2 | 创建第 17 个邻居（超规格） | R-REL5-02 | NEIGHBOR-COUNT=17 | POST 返回错误，拒绝创建 | TP-Q-PERF-002 |
| 3 | 注入 5K BGP4+ 路由 | R-REL5-03 | ROUTE-CAPACITY=5K | 全部学习成功，无内存溢出 | TP-Q-PERF-005 |
| 4 | 注入 50K BGP4+ 路由 | R-REL5-04 | ROUTE-CAPACITY=50K | 全部学习成功，内存增长<20% | TP-Q-PERF-006 |
| 5 | 注入 160K BGP4+ 路由 | R-REL5-05 | ROUTE-CAPACITY=160K | 全部学习成功，无内存溢出 | TP-Q-PERF-007 |
| 6 | 邻居断开触发收敛，测量 ≤5s | R-REL5-06 | CONVERGE-TIME=≤5s | 收敛时间达标 | TP-Q-PERF-003 |
| 7 | 新建邻居，测量 POST 到 Established 时间 ≤5s | R-REL5-08 | ESTABLISH-TIME=≤5s | 建立时间达标 | TP-Q-PERF-004 |
| 8 | 收敛时间 >5s 场景 | R-REL5-07 | CONVERGE-TIME=>5s | 判定不达标 | TP-Q-PERF-003 (扩展) |
| 9 | 建立时间 >5s 场景 | R-REL5-09 | ESTABLISH-TIME=>5s | 判定不达标 | TP-Q-PERF-004 (扩展) |
| 10 | 16 邻居 + 160K 路由 + 线速流量组合压力 | R-REL5-10 | STRESS-MODE=邻居+路由组合 | 邻居保持 + 路由不丢 + 资源不超限 | 组合压力 (P8) |
| 11 | 16 邻居各宣告 100 条路由 + 1h 长稳 | R-REL5-11 | STRESS-MODE=邻居+路由组合 | 1600 条路由正确，内存 CPU 稳定 | 组合压力 (P5) |

## 七、覆盖策略

### 7.1 维度覆盖

| 覆盖维度 | 覆盖方式 | 覆盖规则 | 边界说明 |
|----------|----------|----------|----------|
| 邻居规格 | 边界值: 16 (上限) + 17 (越界) | R1, R2 | 等价类: 规格内 / 超规格 |
| 路由容量 | 三档递增: 5K / 50K / 160K | R3, R4, R5 | 覆盖全系列到旗舰设备规格 |
| 收敛时间 | 达标 / 不达标对偶 | R6, R7 | 阈值 ≤5s |
| 建立时间 | 达标 / 不达标对偶 | R8, R9 | 阈值 ≤5s |
| 组合压力 | 极限组合 + 中等组合长稳 | R10, R11 | 多因子叠加验证系统稳定性 |

### 7.2 测试点覆盖矩阵

| 测试点 | 描述 | 覆盖规则 |
|--------|------|----------|
| TP-Q-PERF-001 | 邻居规格上限验证 (16 个) | R-REL5-01 |
| TP-Q-PERF-002 | 邻居超规格验证 (17 个) | R-REL5-02 |
| TP-Q-PERF-003 | 收敛时间达标验证 (≤5s) | R-REL5-06, R-REL5-07 |
| TP-Q-PERF-004 | 邻居建立时间达标验证 (≤5s) | R-REL5-08, R-REL5-09 |
| TP-Q-PERF-005 | 路由容量 5K 验证 | R-REL5-03 |
| TP-Q-PERF-006 | 路由容量 50K 验证 | R-REL5-04 |
| TP-Q-PERF-007 | 路由容量 160K 验证 | R-REL5-05 |

### 7.3 待确认项

1. **R-REL5-07 实际测量方法**: 收敛时间 >5s 的不达标判定依赖实际测量工具和测量点定义，需与开发确认测量方案。
2. **设备类型映射**: 5K/50K/160K 三种路由容量规格对应的具体设备型号需从产品规格表中确认。
3. **线速流量定义**: R10 中 "TG 线速流量" 的具体速率和报文大小需在测试执行前明确。
4. **内存/CPU 阈值**: "可接受范围" 和 "不超限" 的具体数值需从系统资源基线中获取。
