# PTM Team MVP 范围

> 版本：v1.0 · 更新：2026-06-08

---

## MVP 定义

**Step 1 MVP**：完成三个核心 Agent（ptm-tde / ptm-te / ptm-tae）的最小可用版本，跑通「用例设计 → 用例执行 → 自动化沉淀」最小闭环。

---

## In Scope（MVP 交付范围）

### ptm-tde ✅ 已交付

| 能力 | 覆盖 | 状态 |
|---|---|---|
| 需求文档导入与解析（feature-parser） | Markdown/Word | ✅ |
| 场景发现（scenario-discovery） | 三层场景 | ✅ |
| M 分析（m-analyzer v3.0） | 10 步 + PPDCS 标注 + atomic-ops + 因子库 | ✅ |
| F 分析（f-analyzer v3.0） | 9 步 + Excel/场景/代码耦合 | ✅ |
| Q 分析（q-analyzer v3.0） | 6 步逐 TSP HTSM 评估 | ✅ |
| PPDCS 五方法设计（8 Skill） | P/P/D/C/S 全覆盖 | ✅ |
| 覆盖率验证 + 交付渲染 | 双层验证 + 16 列物理用例 | ✅ |
| 5 个人工检查点 | GATE-1~5 | ✅ |
| AskUserQuestion 交互 | Claude Code 平台 | ✅ |

### ptm-tae（Step 1 基础设施）🔄 进行中

| 能力 | 覆盖 | 状态 |
|---|---|---|
| 原子能力框架（atomic-ops CLI） | Python CLI + YAML schema + 79 ops | ✅ |
| 防火墙配置原子能力 | 安全策略增删改查 | ✅ |
| 自动化工厂（auto_factory_agent） | 用例库管理 + 回归执行 | 🔄 38% |
| 设备安装 Skill | PXE 裸机安装 | 🔄 初版 |

### ptm-te ⬜ 待启动

| 能力 | 覆盖 | 状态 |
|---|---|---|
| 用例解析与执行 Skill | 解析→配置→打流→判定 | ⬜ |
| 结构化执行日志 | 配置+流量+结果+异常上下文 | ⬜ |

---

## Out of Scope（MVP 不包含）

| 项目 | 原因 | 目标 Step |
|---|---|---|
| ptm-tm 测试经理 Agent | 需等三 Agent 闭环跑通后再开发调度层 | Step 3 |
| ptm-tse 测试架构师 Agent | 需求分析和策略依赖 tde 成熟度 | Step 3 |
| ptm-qa 质量工程师 Agent | 质量度量需积累足够执行数据 | Step 3 |
| 全自动调度（tm→全链路） | 需六 Agent 全部就绪 | Step 4 |
| 应用层流量模拟（HTTP/FTP/DNS 等） | Step 1 先覆盖 L2-L4 层 | Step 2 |
| 禅道全集成（自动创建/回写） | Step 2 人工主导阶段先简化 | Step 2 |
| WPS/文档在线协作 | 非核心路径 | Step 2 |

---

## Deferred（延后但保留）

| 项目 | 跟踪 |
|---|---|
| CR-011-T-01 断点恢复机制 | `process/changes/CR-011-FOLLOW-UP-TRACKING-2026-06-02.md` |
| CR-011-T-02 关键词调优 | 同上 |
| CR-015-T-01 Codex 平台整改 | `process/changes/CR-015-FOLLOW-UP-TRACKING-2026-06-04.md` |
| atomic-ops aliases 补充（47 ops） | CR-016 spike_candidate |
| 因子库扩展（安全策略/NAT/HA 因子） | `docs/ptm-team-blueprint.md` §4.2.2 |

---

## MVP 成功标准

| # | 标准 | 度量 | 状态 |
|---|---|---|---|
| 1 | ptm-tde 可独立完成从需求到用例的全流程 | 12 步流程 5 个 Gate 全部通过 | ✅ |
| 2 | ptm-tae 提供可用的原子能力和自动化工厂 | CLI 79 ops + 工厂 4/6 Skill 可用 | 🔄 |
| 3 | ptm-te 可解析用例并调用原子能力执行 | 一次完整的解析→配置→打流→判定闭环 | ⬜ |
| 4 | tde → te → tae 最小闭环可运行 | 端到端：用例设计→执行→执行记录→自动化翻译 | ⬜ |
