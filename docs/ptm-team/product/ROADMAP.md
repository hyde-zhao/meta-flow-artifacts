# PTM Team 路标与阶段

> 版本：v1.0 · 创建：2026-07-06 · 维护：host-orchestrator
> 本文件是 ptm-team 项目演进路标的**单一权威视图**，整合四阶段路线、发布切片、开发里程碑和当前位置。

---

## 文档定位

项目路标信息原本散落于以下文档，本文件作为索引与权威视图整合呈现；各文档保留自身细节，不因本文件而废弃。

| 维度 | 详细文档 |
|------|---------|
| 四阶段演进路线（含各阶段任务表） | `docs/ptm-team-blueprint.md` §1.4 |
| 按 Step 分层的 Story Map | `docs/product/STORY-MAP.md` |
| 发布切片（版本与 CR 映射） | `docs/product/RELEASE-SLICES.md` |
| Step 1 MVP 范围与成功标准 | `docs/product/MVP-SCOPE.md` |
| Slice 1 开发里程碑 M1-M6 | `process/DEVELOPMENT-PLAN.yaml` |
| 当前工作流状态 | `process/STATE.md` |

---

## 一、四阶段演进路线总览

```
Step 1                   Step 2                   Step 3                   Step 4
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│    准备阶段       │    │    人工主导       │    │   Agent 主导     │    │   Autopilot     │
│    Copilot       │───►│    Copilot       │───►│    Copilot       │───►│   全自动         │
│    基础设施       │    │    辅助执行       │    │    自主调度       │    │   数字员工       │
└──────────────────┘    └──────────────────┘    └──────────────────┘    └──────────────────┘
      当前阶段

Agent 就绪:
tde ✓ + te 🔄 + tae 🔄    三大 Agent 持续优化       +tm +tse +qa 补齐六大     全自动闭环
```

| Step | 名称 | 版本 / Slice | 核心转变 | 当前状态 |
|------|------|------------|---------|---------|
| Step 1 | 准备阶段：Copilot 基础设施 | v1.0 / Slice 1 | 0→1 基础设施搭建 | **当前** |
| Step 2 | 人工主导：Copilot 辅助执行 | v1.1 / Slice 2 | 三 Agent 全链路闭环 | 规划中 |
| Step 3 | Agent 主导：Copilot 自主调度 | v2.0 / Slice 3 | 六 Agent 自主协同 | 规划中 |
| Step 4 | Autopilot：全自动数字员工 | v3.0 / Slice 4 | 端到端零人工干预 | 规划中 |

---

## 二、各阶段目标

### Step 1 — 准备阶段：Copilot 基础设施（当前）

**目标**：搭建三大 Agent（ptm-tde / ptm-te / ptm-tae）+ 六大支柱，完成从 0 到 1 的基础设施建设。

| 支柱 | 目标 |
|------|------|
| **因子库** | 公共因子库（基础 + 跨特性通用）全部建设 + 至少 1 个专有特性因子库（如 PBR）跑通完整流程；YAML schema 统一，ptm-tde 设计格式、ptm-tae 实现解析、ptm-te 执行积累 |
| **原子操作** | `atomic-ops` CLI 覆盖防火墙常用特性 REST API（安全策略/NAT/接口/路由/VPN 等，增删改查+批量+回滚）+ 工具原子操作（数通/安全仪表/环境管理）；`{device}_{verb}_{target}` 命名，14 错误码 |
| **耦合矩阵** | 耦合关系数据模型定稿（模块节点/耦合边/类型/影响范围）+ 第一轮代码静态分析，覆盖核心模块 80%+ 已知耦合 |
| **Wiki 体系** | 私人 Wiki 可用可搜索，与公共 Wiki 引用联通，隐私数据不进公共检索 |
| **Agent 交付** | ptm-tde：12 步主流程全 Skill 开发，HLD+REQUIREMENTS 定稿，9 Story 全 verified；ptm-te：功能与架构设计定稿，核心 Skill 初版；ptm-tae：4 阶段方案定稿，公共 Skill 与工具框架就绪 |

**Agent 就绪**：tde ✅ + te 🔄 + tae 🔄
**前置依赖**：无（项目起点）
**退出标准**：三核心 Agent 最小闭环可运行（tde 设计 → te 执行 → tae 翻译沉淀）

### Step 2 — 人工主导：Copilot 辅助执行

**目标**：人工掌握流程控制权，Agent 辅助执行，打通「需求分析 → 用例设计 → 任务执行 → 自动化翻译 → 回归运行」全链路。

| 维度 | 目标 |
|------|------|
| **因子库** | 扩展到全部防火墙特性（安全策略/NAT/HA/VPN/PPPoE/IPv6/SD-WAN），每特性覆盖等价类/边界值/组合约束/正反异常，支持 Pairwise |
| **原子操作** | 全特性配置接口覆盖，工具原子操作生产可用，任一特性执行无需手工 WebUI/CLI |
| **耦合矩阵** | 进入持续维护，与代码仓库同步，F 分析可直接消费最新耦合数据 |
| **Wiki 体系** | 公共 Wiki 覆盖全部特性（描述/配置手册/问题定位/测试分析/已知缺陷），成为团队统一检索入口 |
| **Agent 交付** | tde/te/tae 专属 Skill 全开发，三链路（设计/执行/翻译）稳定运行；tm/tse/qa 管理 Agent 功能能力建设完成（初版） |

**前置依赖**：ptm-tae 原子能力 + Topo 管理就绪
**退出标准**：人工调用 Agent 可完成全部测试工作；管理 Agent 具备基础功能，为 Step 3 自主调度打基础

### Step 3 — Agent 主导：Copilot 自主调度

**目标**：以 ptm-tm 为调度核心，六大 Agent 形成自主协同体系，人工退居关键决策点和质量闸门确认。

| 维度 | 目标 |
|------|------|
| **自动调度** | tm 自动截取禅道任务 → 制定计划 → 调度 tse 需求分析 → 调度 tde 用例设计 → 调度 te/tae 执行沉淀 → 调度 qa 质量评估，全流程 Agent 自主 |
| **调度策略** | 基于任务优先级、Agent 负载、特性风险等级自动分配资源，多项目/多版本并行不冲突 |
| **异常处理** | tae 自动修复可识别脚本失败；te 执行异常自动调用 debug-skill 初步定位；已知异常自动闭环，仅未知异常升级人工 |
| **人工介入** | 仅 4 个关键闸门：tm 计划 / tse 策略 / tde 用例 / qa 质量报告 |

**前置依赖**：Slice 2 闭环跑通
**退出标准**：从任务接入到报告输出全流程由 Agent 自主调度完成

### Step 4 — Autopilot：全自动数字员工

**目标**：六大 Agent 全部完成开发，形成全自动闭环，端到端零人工干预，人工从「操作者」「调度者」彻底转变为「监督者」。

| 维度 | 目标 |
|------|------|
| **全自动运行** | 自动监听禅道/版本转测通知 → 自动完成需求分析/用例设计/测试执行/自动化沉淀/质量评估 → 自动输出报告推送钉钉 |
| **质量自愈** | qa 自动评估过程与产品质量、识别退化趋势；tae 自动修复失败脚本，修复失败自动升级人工 |
| **持续演进** | 缺陷模式自动补充因子库，耦合关系自动更新矩阵，问题定位经验自动沉淀 Wiki |
| **人工职责** | 每日查看执行摘要/风险看板、处理无法自修复异常、审核关键交付物、优化元提示词与元工作流 |
| **最终愿景** | 每位工程师拥有自己的数字员工；公共 Skill/工具作为共享基础设施（ptm-tae 维护），个人元提示词/元工作流作为私有扩展 |

**前置依赖**：Slice 3 六 Agent 协同成熟
**退出标准**：端到端零人工干预，测试全流程自动化运转

> **设计原则贯穿四阶段**：渐进落地（Skill → 工作流 → Agent，单点跑通再组合）、分层解耦（规划层/执行层/感知层独立演进）、共建共用（公共 Skill 与工具由 ptm-tae 统一维护）、AI 原生（框架同时服务 AI 自治和手工测试辅助）。

---

## 三、Slice 1 内部开发里程碑（M1-M6）

ptm-tde v1.0 的 6 个 Wave / 9 个 Story 开发里程碑（定义于 `process/DEVELOPMENT-PLAN.yaml`），全部 verified：

| 里程碑 | Wave | Story | 目标 | 状态 |
|--------|------|-------|------|------|
| **M1** | W1+W2 | STORY-01/02/03 | 安装基线（Claude/Codex project+user scope 投影）+ 场景基础（场景链/动作源/只读 MCP） | ✅ verified |
| **M2** | W2 | STORY-04 | MFQ Trace Chain 贯通（M/F/Q/Integrator 新模型，TP/LC 回链 Scenario Chain） | ✅ verified |
| **M3** | W3 | STORY-05 | 设计计划层 design-planner（CAE→PPDCS 完整推断与设计计划） | ✅ verified |
| **M4** | W4 | STORY-06/07 | 五类设计方法成组落地（process/state-design + parameter/data/combination） | ✅ verified |
| **M5** | W5 | STORY-08 | 覆盖、交付与检索（coverage-verifier / deliverable-renderer / case-retriever） | ✅ verified |
| **M6** | W6 | STORY-09 | 反馈驱动链路（change-impact-analyzer / bug-gap-analyzer 消费 Scenario Chain） | ✅ verified |

---

## 四、当前位置（截至 2026-07-06）

| 维度 | 状态 |
|------|------|
| 项目路标 | **Step 1**（准备阶段） |
| 发布切片 | Slice 1 (v1.0) 已发布（2026-06-06，`release_decision=READY`，`release_artifact_profile=compact`） |
| ptm-tde | v1.0 delivered（CR-010~017，23 Stories，M1-M6 全部 verified） |
| ptm-tae | Step 1 进行中（原子能力框架 ✅ 79 ops，自动化工厂 38%，设备安装初版） |
| ptm-te | Step 1 待启动 |
| active CR | CR-018（流程合规 + 安装投影 + 多特性工作区隔离，implementation 阶段） |

---

## 五、概念区分：工作流 phase vs 项目路标

| 概念 | 含义 | 来源 |
|------|------|------|
| **项目路标**（Step 1-4） | ptm-team 整体演进阶段 | 本文件 + `blueprint §1.4` |
| **meta-flow 工作流 phase** | CP0-CP8 流程阶段（如 `delivered`） | `process/STATE.md` `current_phase` |

`process/STATE.md` 的 `current_phase: delivered` 表示 ptm-tde v1.0 这一轮交付的 meta-flow 工作流走到终验，**不是项目路标**。两者层级不同，不要混淆：
- 工作流 phase = 单轮交付（如 ptm-tde v1.0）在 meta-flow 流程中的位置
- 项目路标 = ptm-team 整体四阶段演进中的位置

---

## 修订记录

| 日期 | 版本 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 2026-07-06 | v1.0 | host-orchestrator | 创建路标文档，整合 `blueprint §1.4` + `STORY-MAP` + `RELEASE-SLICES` + `DEVELOPMENT-PLAN` 的路标信息为单一权威视图；补 Slice 1 M1-M6 里程碑表与当前位置 |
