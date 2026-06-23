# BGP4+ P-Parameter 判定表分析 — LC-REL-001: 进程级可靠性

> 生成时间: 2026-06-12 | 方法: parameter-design (PPDCS P-Parameter)
> design-plan.md / design-planner-reasoning.md: 缺失

## 一、设计计划与推理摘要

> **注意**: design-plan.md 和 design-planner-reasoning.md 均不存在于项目仓库中，因此本节内容基于 logic-cases.md 中 LC-REL-001 的直接描述以及 P-Parameter 方法论推导。

### 1.1 设计目标

对 LC-REL-001「BGP4+ 进程级可靠性」执行 P-Parameter 判定表分析，将进程类型、状态前置条件、Watchfrr 守护状态和恢复时间阈值组合为参数化判定规则，输出覆盖正常路径与异常/边界路径的完整判定表。

### 1.2 分析方法

- **方法**: P-Parameter (PPDCS)，适用于「进程类型 × 恢复时间 × 守护状态」的多因子规则判定场景。
- **CAE 聚合规则**: 规则1-参数化 —— 用规则形式表达因子间的参数化关系，而非笛卡尔积全组合。
- **判定维度来源**: 从 logic-cases.md 提取「被kill进程(bgpd/frr) → 恢复时间≤5s」的核心判定链。

### 1.3 输入来源

| 输入项 | 来源 |
|--------|------|
| 逻辑用例定义 | logic-cases.md LC-REL-001 |
| 覆盖测试点 | TP-Q-REL-001(bgpd), TP-Q-REL-002(FRR) |
| 关联需求 | SR-002, SR-010 |
| 设计计划 | **缺失** — design-plan.md 不存在 |
| 设计推理 | **缺失** — design-planner-reasoning.md 不存在 |

### 1.4 关键假设

1. Watchfrr 是 FRR 套件中的守护进程监控组件，负责检测 bgpd/frr 异常并触发自动重启。
2. 恢复时间阈值 ≤5s 来源于 SR-002/SR-010，但具体测量起止点（从 kill 信号发出到邻居 Established）待确认。
3. "恢复"的完整定义包含两个阶段：进程拉起 + BGP 邻居状态重建至 Established + 路由信息恢复。

## 二、Factor Catalog

### 2.1 因子总览

| 因子 ID | 因子名称 | 数据类型 | 角色 | 取值域 |
|---------|----------|----------|------|--------|
| FAC-BGP6-KILL-PROCESS | 被kill进程 | enum | control, driver | bgpd, frr |
| FAC-BGP6-PRE-STATE | kill前状态 | enum | condition | Established+路由已通告, 仅Enabled无邻居 |
| FAC-BGP6-RECOVERY-TIME | 恢复时间阈值 | data | boundary | ≤5s, 5-15s, >15s |
| FAC-BGP6-WATCHFRR | Watchfrr守护进程状态 | enum | condition | 正常运行, 异常/停止 |

### 2.2 因子详细说明

#### FAC-BGP6-KILL-PROCESS: 被kill进程

- **角色**: control, driver
- **取值**: `bgpd` | `frr`
- **说明**: 
  - `bgpd`: 仅 BGP 守护进程被 kill，FRR 管理框架仍在运行。
  - `frr`: 整个 FRR 进程被 kill，所有路由协议守护进程（包括 bgpd）均受影响。
- **驱动关系**: 该因子是判定表的主驱动轴，直接决定恢复路径差异。

#### FAC-BGP6-PRE-STATE: kill前状态

- **角色**: condition
- **取值**: `Established+路由已通告` | `仅Enabled无邻居`
- **说明**: 
  - `Established+路由已通告`: kill 前 BGP 邻居已建立且路由信息已通告，恢复后需验证邻居重建和路由恢复两个维度。
  - `仅Enabled无邻居`: kill 前 BGP 已启用但尚未建立邻居，恢复后仅需验证进程拉起和配置加载。

#### FAC-BGP6-RECOVERY-TIME: 恢复时间阈值

- **角色**: data, boundary
- **取值**: `≤5s` | `5-15s` | `>15s`
- **说明**: 
  - `≤5s` 为通过标准（来源于 SR-002/SR-010）。
  - `5-15s` 为灰色区间，需根据实际产品规格确认是否可接受。
  - `>15s` 为明确不达标。
- **边界关注点**: 5s 边界附近的测量精度和计时起止点定义。

#### FAC-BGP6-WATCHFRR: Watchfrr守护进程状态

- **角色**: condition
- **取值**: `正常运行` | `异常/停止`
- **说明**: 
  - `正常运行`: Watchfrr 可正常检测进程异常并触发自动恢复。
  - `异常/停止`: Watchfrr 自身故障，无法触发自动恢复，需人工介入验证。

## 三、Topology Binding Catalog

### 3.1 拓扑绑定说明

LC-REL-001 为进程级可靠性测试，不依赖特定拓扑结构。核心关注点为单节点 FRR 实例的进程生命周期管理。

| 绑定项 | 绑定内容 | 说明 |
|--------|----------|------|
| 节点类型 | 单节点 DUT（运行 FRR/bgpd） | 进程级别，不涉及多节点交互 |
| 邻居拓扑 | 至少 1 个 BGP Peer（可简化） | 用于验证邻居重建和路由恢复 |
| 配置来源 | frrinc.conf | FRR 集成配置文件，kill frr 后从此文件恢复 |
| 测量工具 | kill 命令 + systemd/supervisor 监控 | 进程 kill 和拉起观测 |

### 3.2 Topology Binding 映射

| 规则 ID | 节点要求 | BGP Peer 要求 | 特殊配置 |
|---------|----------|---------------|----------|
| R-REL1-01 | 1× DUT | ≥1 Peer (Established) | bgpd 由 Watchfrr 管理 |
| R-REL1-02 | 1× DUT | ≥1 Peer | frrinc.conf 正确配置 |
| R-REL1-03 | 1× DUT | ≥1 Peer | 时间测量工具 |
| R-REL1-04 | 1× DUT | ≥1 Peer | Watchfrr 可控停止 |

## 四、参数范围与规则清单

### 4.1 参数范围界定

| 参数 | 覆盖取值 | 未覆盖取值 | 原因 |
|------|----------|------------|------|
| FAC-BGP6-KILL-PROCESS | bgpd, frr | 其他 FRR 组件(ospfd, zebra等) | 需求限定为 bgpd/FRR 进程级，其他组件不在 LC-REL-001 范围内 |
| FAC-BGP6-PRE-STATE | Established+路由已通告 | 仅Enabled无邻居 | 主路径使用 Established 状态；无邻居场景可作为扩展但非核心 |
| FAC-BGP6-RECOVERY-TIME | ≤5s, >5s | 5-15s, >15s 细分 | 判定边界为 5s，细分仅用于边界分析 |
| FAC-BGP6-WATCHFRR | 正常运行, 异常/停止 | — | 二元条件全覆盖 |

### 4.2 规则清单

#### R-REL1-01: bgpd 正常恢复（主路径）

- **条件**: 
  - FAC-BGP6-KILL-PROCESS = `bgpd`
  - FAC-BGP6-PRE-STATE = `Established+路由已通告`
  - FAC-BGP6-WATCHFRR = `正常运行`
- **预期结果**: 
  - Watchfrr 检测到 bgpd 异常 → 自动拉起 bgpd
  - bgpd 从内存/配置文件恢复运行状态
  - BGP 邻居重建至 Established
  - 路由信息恢复通告
  - FAC-BGP6-RECOVERY-TIME ≤ 5s
- **trace_refs**: TP-Q-REL-001
- **priority**: P1
- **fact_status**: confirmed

#### R-REL1-02: FRR 整体恢复（主路径）

- **条件**: 
  - FAC-BGP6-KILL-PROCESS = `frr`
  - FAC-BGP6-WATCHFRR = `正常运行`
- **预期结果**: 
  - Watchfrr 检测到 FRR 异常 → 自动拉起 FRR
  - FRR 从 frrinc.conf 恢复所有配置
  - 所有路由协议守护进程（包括 bgpd）重新初始化
  - BGP 邻居重建至 Established
- **trace_refs**: TP-Q-REL-002
- **priority**: P1
- **fact_status**: confirmed

#### R-REL1-03: bgpd 恢复超时（边界路径）

- **条件**: 
  - FAC-BGP6-KILL-PROCESS = `bgpd`
  - FAC-BGP6-PRE-STATE = `Established+路由已通告`
  - FAC-BGP6-WATCHFRR = `正常运行`
  - FAC-BGP6-RECOVERY-TIME > 5s
- **预期结果**: 
  - bgpd 最终恢复成功但恢复时间超过 5s 阈值
  - 可靠性指标不达标
  - **[待确认]**: 阈值来源（SR-002/SR-010 的具体指标），以及 >5s 时是否有降级处理策略
- **trace_refs**: 扩展（由 R-REL1-01 边界衍生）
- **priority**: P2
- **fact_status**: needs-confirmation

#### R-REL1-04: Watchfrr 异常无法自动恢复（异常路径）

- **条件**: 
  - FAC-BGP6-KILL-PROCESS = `bgpd`
  - FAC-BGP6-PRE-STATE = `Established+路由已通告`
  - FAC-BGP6-WATCHFRR = `异常/停止`
- **预期结果**: 
  - kill bgpd 后 Watchfrr 无法检测异常
  - bgpd 不会自动拉起
  - BGP 邻居中断，路由不可达
  - 需人工介入恢复
  - **[待确认]**: 此场景下系统是否有告警机制？是否有 fallback 监控手段？
- **trace_refs**: 扩展（Watchfrr 异常场景）
- **priority**: P2
- **fact_status**: needs-confirmation

## 五、判定结构（判定表）

| 条件/动作 | R1 (R-REL1-01) | R2 (R-REL1-02) | R3 (R-REL1-03) | R4 (R-REL1-04) |
|-----------|----------------|----------------|----------------|----------------|
| FAC-BGP6-KILL-PROCESS | bgpd | frr | bgpd | bgpd |
| FAC-BGP6-PRE-STATE | Established+路由已通告 | Established+路由已通告 | Established+路由已通告 | Established+路由已通告 |
| FAC-BGP6-WATCHFRR | 正常运行 | 正常运行 | 正常运行 | 异常/停止 |
| FAC-BGP6-RECOVERY-TIME | ≤5s | ≤5s | >5s | N/A (不恢复) |
| **恢复动作** | Watchfrr自动拉起bgpd | Watchfrr自动拉起FRR | Watchfrr自动拉起bgpd | 无自动拉起 |
| **邻居结果** | 重建至Established | 重建至Established | 重建至Established | 中断 |
| **路由结果** | 路由恢复通告 | 路由恢复通告 | 路由恢复通告 | 路由不可达 |
| **判定** | 通过 | 通过 | 不达标（超时） | 不达标（无恢复） |
| **trace_refs** | TP-Q-REL-001 | TP-Q-REL-002 | 扩展 | 扩展 |
| **priority** | P1 | P1 | P2 | P2 |
| **fact_status** | confirmed | confirmed | needs-confirmation | needs-confirmation |

### 5.1 规则覆盖关系

```
                    ┌──────────────────────────────┐
                    │     FAC-BGP6-KILL-PROCESS     │
                    └──────────┬───────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                │                ▼
         kill bgpd             │           kill frr
              │                │                │
     ┌────────┴────────┐       │        ┌───────┴───────┐
     ▼                 ▼       │        ▼               ▼
  Watchfrr正常      Watchfrr异常│    Watchfrr正常    Watchfrr异常
     │                 │       │        │               │
  ┌──┴──┐           R-REL1-04  │    R-REL1-02      [扩展]
  ▼     ▼            (P2)      │     (P1)           (P2)
 ≤5s   >5s                     │
  │     │                      │
R-REL1-01  R-REL1-03           │
 (P1)       (P2)               │
```

## 六、Data Row → LC 叠加

### 6.1 Data Row 清单

| Data Row ID | 规则映射 | 场景描述 | 输入条件 | 预期结果 | 关联 LC |
|-------------|----------|----------|----------|----------|---------|
| DR-REL-001-01 | R-REL1-01 | kill bgpd → Watchfrr自动拉起 → 邻居重建 → 路由恢复 | kill_proc=bgpd, pre_state=Established+路由, watchfrr=正常 | 恢复时间≤5s, 邻居Established, 路由恢复 | LC-REL-001 |
| DR-REL-001-02 | R-REL1-02 | kill FRR → Watchfrr自动拉起 → 从frrinc.conf恢复配置 → 邻居重建 | kill_proc=frr, pre_state=Established+路由, watchfrr=正常 | FRR拉起成功, 配置恢复, 邻居Established | LC-REL-001 |
| DR-REL-001-03 | R-REL1-03 | kill bgpd → 恢复时间>5s（边界测试） | kill_proc=bgpd, pre_state=Established+路由, watchfrr=正常 | bgpd恢复成功但>5s, 可靠性不达标 | LC-REL-001 |
| DR-REL-001-04 | R-REL1-04 | Watchfrr异常 → kill bgpd后不自动恢复 | kill_proc=bgpd, pre_state=Established+路由, watchfrr=异常 | bgpd未拉起, 邻居中断, 路由不可达 | LC-REL-001 |

### 6.2 Data Row 与测试点映射

| Data Row ID | 测试点 | 覆盖维度 |
|-------------|--------|----------|
| DR-REL-001-01 | TP-Q-REL-001 | bgpd 正常恢复（时间 + 邻居 + 路由）|
| DR-REL-001-02 | TP-Q-REL-002 | FRR 整体恢复（配置 + 邻居）|
| DR-REL-001-03 | TP-Q-REL-001 (扩展) | bgpd 恢复时间边界 |
| DR-REL-001-04 | TP-Q-REL-001 (扩展) | Watchfrr 异常场景 |

### 6.3 Data Row 叠加 LC

```
LC-REL-001: 进程级可靠性
├── DR-REL-001-01 ──→ R-REL1-01 ──→ TP-Q-REL-001 (bgpd 正常恢复)
├── DR-REL-001-02 ──→ R-REL1-02 ──→ TP-Q-REL-002 (FRR 整体恢复)
├── DR-REL-001-03 ──→ R-REL1-03 ──→ 扩展 (时间边界)
└── DR-REL-001-04 ──→ R-REL1-04 ──→ 扩展 (Watchfrr 异常)
```

## 七、覆盖策略

### 7.1 覆盖矩阵

| 覆盖维度 | 取值/场景 | 覆盖规则 | 覆盖状态 |
|----------|-----------|----------|----------|
| 被kill进程 | bgpd | R-REL1-01, R-REL1-03, R-REL1-04 | 已覆盖 |
| 被kill进程 | frr | R-REL1-02 | 已覆盖 |
| kill前状态 | Established+路由已通告 | 全部规则 | 已覆盖 |
| kill前状态 | 仅Enabled无邻居 | — | 未覆盖（非核心）|
| Watchfrr状态 | 正常运行 | R-REL1-01, R-REL1-02, R-REL1-03 | 已覆盖 |
| Watchfrr状态 | 异常/停止 | R-REL1-04 | 已覆盖 |
| 恢复时间 | ≤5s | R-REL1-01, R-REL1-02 | 已覆盖 |
| 恢复时间 | >5s | R-REL1-03 | 已覆盖 |
| 恢复结果 | 邻居重建+路由恢复 | R-REL1-01, R-REL1-02 | 已覆盖 |
| 恢复结果 | 仅邻居重建 | — | 未覆盖（待确认是否存在此中间态）|
| 恢复结果 | 失败 | R-REL1-04 | 已覆盖 |

### 7.2 覆盖度评估

| 评估项 | 结论 |
|--------|------|
| 进程类型覆盖 | 2/2 (bgpd, frr) — 完整 |
| 正常路径覆盖 | 2/2 (bgpd恢复, FRR恢复) — 完整 |
| 边界路径覆盖 | 1/1 (时间边界) — 完整 |
| 异常路径覆盖 | 1/1 (Watchfrr异常) — 完整 |
| 未覆盖风险 | `仅Enabled无邻居` 场景和 `仅邻居重建无路由恢复` 中间态未覆盖，但这两种场景在产品定义中的存在性待确认 |

### 7.3 待确认事项汇总

| 编号 | 事项 | 关联规则 | 优先级 |
|------|------|----------|--------|
| C-001 | 恢复时间阈值 5s 的具体来源（SR-002/SR-010 原文表述）及测量起止点定义 | R-REL1-01, R-REL1-03 | 高 |
| C-002 | >5s 恢复是否有降级处理策略 | R-REL1-03 | 中 |
| C-003 | Watchfrr 异常时系统是否有告警机制 | R-REL1-04 | 中 |
| C-004 | "仅邻居重建但无路由恢复" 是否为实际可能出现的中间态 | 覆盖策略 | 低 |
| C-005 | design-plan.md 和 design-planner-reasoning.md 缺失，是否需要在后续迭代中补齐 | 整体 | 中 |
