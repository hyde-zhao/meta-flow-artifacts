# PTM Team Story Map

> 版本：v1.0 · 更新：2026-06-08 · 基于四步演进路线

---

## 用户活动全景

```
┌────────────────────────────────────────────────────────────────────┐
│                        PTM Team 测试工作流                           │
├──────────┬──────────┬──────────┬──────────┬──────────┬─────────────┤
│ 项目接入 │ 需求分析 │ 用例设计 │ 用例执行 │ 自动化   │ 质量度量    │
│ ptm-tm   │ ptm-tse  │ ptm-tde  │ ptm-te   │ ptm-tae  │ ptm-qa      │
├──────────┼──────────┼──────────┼──────────┼──────────┼─────────────┤
│          │          │          │          │          │             │
│ Step 4   │ Step 4   │ ✅ 已交付 │ Step 2   │ Step 2   │ Step 4      │
│ Autopilot│ Autopilot│ Step 1   │ Copilot  │ Copilot  │ Autopilot   │
│          │          │ 完成     │ 人工主导  │ 人工主导  │             │
│          │          │          │          │          │             │
└──────────┴──────────┴──────────┴──────────┴──────────┴─────────────┘
```

> **当前进度**（2026-07-06）：ptm-tde ✅ Step 1 已交付；ptm-tae 🔄 Step 1 进行中（38%）；ptm-te ⬜ Step 1 待启动。表中"Step X"表示 Agent 的目标就绪阶段，非当前进度。

---

## 按 Step 分层的 Story Map

### Step 1 — Copilot 准备阶段（当前）

**目标**：搭建 ptm-tde / ptm-te / ptm-tae 三个核心 Agent 及底层工具能力

| 用户活动 | Story | Agent | 状态 |
|---|---|---|---|
| **用例设计** | 三阶段框架搭建（KYM → MFQ → PPDCS） | ptm-tde | ✅ CR-010 |
| | KYM 阶段（GATE-1/2 + 方法论确认） | ptm-tde | ✅ CR-011 |
| | MFQ 阶段（M/F/Q 三维分析） | ptm-tde | ✅ CR-012 |
| | PPDCS 阶段（五方法设计 + GATE-4/5） | ptm-tde | ✅ CR-013 |
| | AskUserQuestion 交互增强 | ptm-tde | ✅ CR-015 |
| | 原子操作集成（atomic-ops CLI + 语义匹配） | ptm-tde | ✅ CR-016 |
| | 因子库发现（自动加载 + 四级分级） | ptm-tde | ✅ CR-017 |
| **基础设施** | 原子能力框架（atomic-ops CLI） | ptm-tae | 🔄 进行中 |
| | 防火墙配置原子能力补齐 | ptm-tae | 🔄 进行中 |
| | 数通仪表打流能力 | ptm-tae | 🔄 进行中 |
| | Topo 管理（逻辑 Topo 规范 + 大网建设） | ptm-tae | 🔄 进行中 |
| | 因子库建设（策略路由因子等） | ptm-tde/ptm-tae | 🔄 部分完成 |
| **工具开发** | 自动化工厂维护（auto_factory_agent） | ptm-tae | 🔄 进行中 |
| | 设备安装 Skill（PXE 裸机安装） | ptm-tae | 🔄 初版完成 |

### Step 2 — Copilot 人工主导

**目标**：人工主导，ptm-tde / ptm-te / ptm-tae 打通全链路

| 用户活动 | Story | Agent | 状态 |
|---|---|---|---|
| **用例执行** | ptm-te Agent 开发（解析→配置→打流→判定） | ptm-te | ⬜ planned |
| | 禅道任务集成（领取+回写） | ptm-te | ⬜ planned |
| | Topo 映射（逻辑→物理自动匹配） | ptm-te/ptm-tae | ⬜ planned |
| **自动化翻译** | 执行记录→Python 脚本翻译 | ptm-tae | ⬜ planned |
| | 脚本验证与工厂注册 | ptm-tae | ⬜ planned |
| | 回归触发 + 失败自修复 | ptm-tae | ⬜ planned |
| **闭环验证** | tde → te → tae 最小闭环端到端 | 跨 Agent | ⬜ planned |

### Step 3 — Copilot Agent 主导

**目标**：开发 ptm-tm / ptm-tse / ptm-qa，六 Agent 协同

| 用户活动 | Story | Agent | 状态 |
|---|---|---|---|
| **项目接入** | ptm-tm Agent 开发（计划+调度+报告） | ptm-tm | ⬜ planned |
| **需求分析** | ptm-tse Agent 开发（需求分析+策略制定） | ptm-tse | ⬜ planned |
| | ptm-tse 评审链（用例/执行/工具） | ptm-tse | ⬜ planned |
| **质量度量** | ptm-qa Agent 开发（审计+采集+报告） | ptm-qa | ⬜ planned |
| **协同调度** | tm→tse→tde→te→qa 全链路调度 | 跨 Agent | ⬜ planned |

### Step 4 — Autopilot

**目标**：全自动运行，人工仅处理异常

| 用户活动 | Story | Agent | 状态 |
|---|---|---|---|
| **全自动闭环** | 禅道自动触发 → 全流程自动执行 | 全部 | ⬜ planned |
| | 异常自动处理（已知异常自修复） | 全部 | ⬜ planned |
| | 人工监督面板（每日执行摘要） | ptm-tm | ⬜ planned |

---

## 统计

| Step | Agent | Stories 完成 | Stories 规划中 |
|---|---|---|---|
| Step 1 | ptm-tde | 7 CR (23 Stories) | 0 |
| Step 1 | ptm-tae | 部分 | 3 |
| Step 1 | ptm-te | 0 | 2 |
| Step 2 | ptm-te/ptm-tae | 0 | 5 |
| Step 3 | ptm-tm/tse/qa | 0 | 4 |
| Step 4 | 全部 | 0 | 3 |
| **合计** | | **23** | **17** |
