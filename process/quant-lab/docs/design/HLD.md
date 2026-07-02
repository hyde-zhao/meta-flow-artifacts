---
status: "current-index"
version: "1.12"
change: "CR-151"
legacy_sources:
  - "process/HLD.md"
  - "process/HLD-DATA-LAKE.md"
  - "process/HLD-QMT-TRADING.md"
current_change_sources:
  - "process/archive/design-cr-docs/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md"
  - "process/archive/design-cr-docs/RUNNER-CORE-MVP-DESIGN-CR126.md"
  - "process/docs/features/strategy-runner-core/DESIGN.md"
  - "process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md"
  - "process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
---

# HLD Current Index

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-07 | meta-po | 新增 HLD 长期索引，保留 legacy HLD 为审计来源 |
| 1.1 | 2026-06-13 | meta-po | 按 CR-046 增补双目标策略交付框架 HLD 入口 |
| 1.2 | 2026-06-23 | host-orchestrator | CR130 收口 runner architecture authority：CR046 降级为 legacy cross-target framework，offline runner implementation authority 指向 `strategy-runner-core`。 |
| 1.3 | 2026-06-23 | host-orchestrator | CR131 将 CR 命名 HLD/设计文档移出默认 design surface，历史入口统一指向 `process/archive/design-cr-docs/`。 |
| 1.4 | 2026-06-24 | host-orchestrator | CR138 按功能域新增 Runner / QMT Gateway Operational Control Plane HLD 入口，避免长期 HLD 以 CR 命名；该 HLD 作为 FEAT-11 / FEAT-12 solution-design 基线。 |
| 1.5 | 2026-06-27 | codex | 登记统一因子目录与查询 CLI 为研究消费层 HLD 增量；不新建旁路 HLD，不扩大数据湖、QMT、simulation 或 live 授权。 |
| 1.6 | 2026-06-27 | codex | 登记高级多因子模型评估为研究消费层 HLD 增量；mature admission 必须引用 FactorModelValidationReport，不新建旁路 HLD。 |
| 1.7 | 2026-06-27 | codex | 登记 Stage 3 目标调整：研究消费层必须输出评估通过的成熟多因子策略；blocked 运行不再可作为 Stage 3 完成物。 |
| 1.8 | 2026-06-27 | codex | 登记异象发现与异象研究为研究消费层 HLD 增量；新增 AnomalyCandidate / AnomalyResearchReport / AnomalyAdmissionDecision，不新建旁路 HLD。 |
| 1.9 | 2026-06-27 | codex | 登记自动异象发现为 FEAT-03 研究消费层增量；采用受控模板、批量研究 runner、多重检验、动态目录接入和 Stage 3 候选消费，不新建旁路 HLD。 |
| 1.10 | 2026-06-28 | codex | 登记研究引擎稳定模块整改为 FEAT-03 研究消费层增量；领域名模块成为主实现入口，旧 chapter/stage/root 脚本只作兼容或归档，不新建旁路 HLD。 |
| 1.11 | 2026-07-01 | host-orchestrator | 登记 CR151 Strategy Admission Statistical Gate companion HLD；多因子统计准入采用独立 metadata-only contract module，Wave A 只覆盖 FDR/稳健统计/walk-forward/PBO-DSR/statistical gate，不授权 runtime。 |
| 1.12 | 2026-07-01 | host-orchestrator | 同步 CR151 CP3 评审整改：CR151 Wave A 不覆盖 MF-GAP-2 扩展评价统计、MF-GAP-4 regime 分层和 MF-GAP-7 因子相关性聚类（clustering）/ 去重，均后置到 follow-up / CR154。 |

## 定位

本文不是新的完整 HLD，也不替代 `process/HLD*.md`。它是长期阅读入口，用于说明当前项目三类 HLD 的职责分布和消费顺序。

## HLD 分区

| HLD | 职责 | 当前用途 |
|---|---|---|
| `process/HLD.md` | 本地研究、轻量回测、研究消费、报告、多因子闭环、Backtrader optional reference、legacy strategy readiness admission 和 QMT gateway 相关主线索引 | 主研究 / 消费层 HLD legacy source |
| `process/HLD-DATA-LAKE.md` | 生产级市场数据湖、P0/P1 dataset、catalog current truth、publish gate、DuckDB readonly candidate、复权双视图、production current truth closure | 数据生产 / 数据事实源 HLD legacy source |
| `process/HLD-QMT-TRADING.md` | QMT 交易接入、OMS、adapter、broker lake、stage gate、runbook、reconciliation、kill switch、QMT C/S bridge 交易治理侧 | QMT 交易治理 HLD legacy source |
| `process/archive/design-cr-docs/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | QMT terminal + MiniQMT runner 双目标策略交付框架、策略包契约、验证框架、MiniQMT runner 安装设计和后续 CR 门禁 | archived legacy cross-target framework；不再作为 offline runner implementation authority |
| `process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` | Runner Control Plane 与 QMT Gateway Service Layer 的运营控制面 HLD，覆盖 FEAT-11 / FEAT-12、REST-only Gateway P0、按需 runtime 授权、交易日历、佣金和收益查询能力 | 当前 CR138 solution-design 基线；CP3 approved，可作为 CP4 Story planning 输入 |

## 研究消费层增量

| 增量 | 所属 HLD 分区 | 范围 | 不包含 |
|---|---|---|---|
| Unified Factor Catalog / CLI | `process/HLD.md` 研究消费层，FEAT-03 factor-research-loop | 在代码内登记核心 7 因子、mature runner 5 因子和 Chapter5 anomaly proxy 3 因子的身份、来源、状态、计算能力、使用方和证据引用；提供只读查询 CLI；mature runner `FactorSpec` 元数据从目录读取 | 不连接数据湖、不发布 catalog、不迁移 runner-local 计算器、不改变策略收益逻辑、不授权 QMT / simulation / live |
| Advanced Factor Model Validation | `process/HLD.md` 研究消费层，FEAT-03 factor-research-loop；FEAT-14 mature admission 只读消费 | 生成 `factor_model_validation_report_v1`，覆盖 GRS、因子溢价显著性、经济显著性、时间切分、测试资产多样性、样本外、壳价值控制、做空可行性、政策周期覆盖、因子冗余、风格暴露、参数敏感性、IC 衰减、多重检验、数据偏差、组合构建稳健性、容量冲击和尾部风险；mature admission package 必须记录报告引用和 gate summary；Stage 3 完成物必须满足核心门禁无 blocked、mature admission PASS 和 runner offline / preflight PASS | 不连接数据湖、不生产 policy cycle 数据集、不读取 broker shortability facts、不改变 mature runner 组合收益逻辑、不授权 QMT / simulation / live；blocked 运行只能作为诊断基线，不能作为 Stage 4 输入 |
| Anomaly Discovery / Research | `process/HLD.md` 研究消费层，FEAT-03 factor-research-loop；FEAT-14 mature admission 只读消费通过后的异象候选 | 形成理论驱动的 `AnomalyCandidate`，执行单变量排序、Harvey t>=3、五分组单调性、控制现有因子 alpha、时间切分、子样本 / A 股特殊处理、扣费后收益、换手可行性和经济逻辑审查，输出 `AnomalyResearchReport` 与 `AnomalyAdmissionDecision` | 不自动发现生产事实、不把参数搜索胜者直接入库、不授权 QMT / simulation / live；watch / rejected / blocked 异象不得进入 Stage 3 因子候选 |
| Automatic Anomaly Discovery | `process/HLD.md` 研究消费层，FEAT-03 factor-research-loop；Stage 3 candidate search 显式消费 | 从固定 `ControlledAnomalyTemplate` 生成候选，批量运行 Chapter5 风格排序 / alpha / 时间切分 / 成本 / 经济逻辑评估，附加 Bonferroni 与 BH-FDR 多重检验，输出 discovery run、admission decisions 和动态 FactorCatalogEntry extras | 不做黑箱公式搜索、不写数据湖、不 publish catalog、不把一次运行结果硬编码进静态 registry、不授权 QMT / simulation / live |
| Research Engine Stable Module Consolidation | `process/HLD.md` 研究消费层，FEAT-03 factor-research-loop；FEAT-13 / FEAT-14 只读消费稳定入口 | 将旧章节式实现收敛为 `factor_replication`、`research_data_readiness`、`factor_model_research`、`anomaly_research`、`factor_robustness`、`factor_portfolio_practice`；抽取 `serialization`、`factor_research_matrices` 和 `admission_contracts`；旧 engine chapter wrapper 归档到 `docs/legacy/archive/engine/`；稳定脚本入口位于 `scripts/research/*` / `scripts/qmt/*` | 不改变研究算法收益逻辑、不批量重写历史 schema / run_id / report artifact、不授权数据湖 / QMT / simulation / live |
| Strategy Admission Statistical Gate | `process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md`；FEAT-03 / FEAT-14 研究准入层 | 新增多因子 Wave A 统计准入门：MultipleTestingReport、RobustFactorStatisticsReport、WalkForwardValidationPlan、BacktestOverfitRiskReport、StrategyAdmissionStatisticalGate；复用现有 ResearchDatasetSpec / BacktestRunSpec / StrategyAdmissionPackage，不创建平行契约 | 不读真实 lake/NAS/provider，不运行 QMT/simulation/live/trading，不读 credential，不引入外部框架；capacity/impact、IR/TE/Active Share、PIT universe audit、MF-GAP-2 扩展评价统计、MF-GAP-4 regime 分层和 MF-GAP-7 因子相关性聚类（clustering）/ 去重后置到 follow-up / CR154 |

## Runner Architecture Authority

| 层级 | 当前权威入口 | 角色 |
|---|---|---|
| 长期 Feature authority | `process/docs/features/strategy-runner-core/DESIGN.md` | offline runner core 的长期功能边界和实现权威，覆盖 `RunSpec`、package loader 消费、adapter dispatch、fake readonly boundary、evidence summary 和 `RunResult`。 |
| Source design / implementation intake | `process/archive/design-cr-docs/RUNNER-CORE-MVP-DESIGN-CR126.md` | CR126/CR128 runner core MVP 的设计输入；用于追溯实现收口，不替代长期 Feature authority。 |
| Legacy cross-target HLD | `process/archive/design-cr-docs/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | 保留 CR046 QMT terminal + MiniQMT 双目标策略交付历史框架；不得直接恢复 offline runner implementation。 |
| Runtime / NAS / provider / trading gates | 后续独立 CR | 真实 QMT/MiniQMT runtime、NAS package exchange、provider/lake/catalog 和 trading write 均需独立授权门禁。 |
| Runner / QMT Operational Control Plane | `process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` | FEAT-11 Runner Control Plane 与 FEAT-12 QMT Gateway Service Layer 的长期 HLD 基线；CP3 approve 不自动授权 runtime，后续验证按需授权。 |

## 当前架构摘要

| 层 | Owner Feature | 当前边界 |
|---|---|---|
| 数据生产 | FEAT-02 | 只有用户显式授权的数据湖生产链路可触发 provider / lake write / publish；consumer 不自动补数 |
| 研究消费 | FEAT-01 / FEAT-03 | 只读 published data、quality/readiness、claim boundary；输出研究报告、factor catalog、factor evaluation 和 admission package |
| 执行语义 | FEAT-04 | lightweight 为默认主路径；Backtrader / external frameworks 只作 optional reference / Spike |
| QMT gateway | FEAT-05 | 当前 CR-020 只到 Windows/QMT `query_positions` readonly manual validation，不授权交易或 simulation/live |
| 交易治理 | FEAT-06 | CR-015/016 已有 shadow / dry-run / mock / runbook 边界，后续 CR-021..024 才可能逐级解禁 |
| 安全授权 | FEAT-07 | 所有真实操作必须有独立授权；文档、健康检查、Story verified 不构成授权 |
| 双目标策略交付 | FEAT-09 | CR046 保留为 legacy cross-target framework；offline runner implementation authority 已转移到 `strategy-runner-core`；真实 runtime/NAS/QMT 仍需独立门禁 |
| Runner 运营控制面 | FEAT-11 | 日常 run plan、preflight、事件 / 信号、再平衡、run tracking、证据、复盘、incident、策略生命周期；不直连 QMT / XtQuant |
| QMT Gateway 服务层 | FEAT-12 | REST-only P0，覆盖 lifecycle、health、capabilities、reference/account query、market subscription management、order command gate、execution report、recovery、audit、change plan；账户级查询和 runtime 动作按需授权 |

## 消费规则

| 场景 | 默认读取 |
|---|---|
| 判断 Feature 边界 | `docs/design/BLUEPRINT.md` |
| 判断领域对象和状态 | `docs/design/DOMAIN-MAP.md` |
| 判断允许 / 禁止依赖 | `docs/design/DEPENDENCY-MAP.md` |
| 深入 HLD 细节 | 对应 `process/HLD*.md` |
| 审查 CR046 双目标策略交付框架 | `process/archive/design-cr-docs/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md`，仅作为 archived legacy cross-target framework |
| 审查 offline runner implementation authority | `process/docs/features/strategy-runner-core/DESIGN.md` |
| 审查 CR128 runner core source design | `process/archive/design-cr-docs/RUNNER-CORE-MVP-DESIGN-CR126.md` |
| 判断 ADR | `docs/design/ARCHITECTURE-DECISION.md` + `process/ARCHITECTURE-DECISION.md` |
| 判断 Story 范围 | `process/STORY-BACKLOG.md`、`process/stories/*.md` |

## 不授权项

本文不授权 provider fetch、lake write、catalog publish、DuckDB 事实源写入、gateway 启动、端口绑定、QMT / MiniQMT / XtQuant 调用、真实 `.env` 读取、账户查询、交易、撤单、simulation、live_readonly、small_live、scale_up 或 broker lake 写入。
