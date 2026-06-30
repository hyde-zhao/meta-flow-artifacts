---
status: "draft-current-index"
version: "1.14"
source_use_cases: "process/USE-CASES.md"
source_requirements: "process/REQUIREMENTS.md"
source_story_backlog: "process/STORY-BACKLOG.md"
source_hld:
  - "process/HLD.md"
  - "process/HLD-DATA-LAKE.md"
  - "process/HLD-QMT-TRADING.md"
  - "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
  - "process/archive/design-cr-docs/HLD-CR051-STRATEGY-RESEARCH-LIFECYCLE-FRAMEWORK.md"
source_adr: "process/ARCHITECTURE-DECISION.md"
change: "CR-139"
companion_hld_cr139: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
companion_adr_cr139: "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
superseded_in_scope_hld: "process/HLD-DATA-LAKE.md (CR-018, superseded-in-scope by CR-139 companion HLD)"
confirmed_by: ""
confirmed_at: ""
archived_previous:
  - "process/archive/design-blueprints/BLUEPRINT-before-quant-lab-project-roadmap-2026-06-26.md"
  - "process/archive/design-blueprints/DOMAIN-MAP-before-quant-lab-project-roadmap-2026-06-26.md"
  - "process/archive/design-blueprints/DEPENDENCY-MAP-before-quant-lab-project-roadmap-2026-06-26.md"
---

# Blueprint

> 本文是 `quant-lab` 的蓝图层 current index；`local_backtest` 保留为 legacy alias 和历史审计名。本文用于归一化跨 Feature / Epic 的能力边界、数据归属、跨模块流程和后续 Feature 设计入口。历史设计细节仍以 `process/HLD*.md`、`process/ARCHITECTURE-DECISION.md` 和 `process/STORY-BACKLOG.md` 为审计来源。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-07 | meta-po | 按 CR-031 新增蓝图层索引，覆盖研究回测、生产数据湖、多因子研究、执行语义、QMT gateway、交易治理、安全授权和文档 Runbook 八个 Feature / Epic |
| 1.1 | 2026-06-13 | meta-po | 按 CR-046 增补 QMT terminal + MiniQMT runner 双目标策略交付框架 Feature、策略包契约、验证框架和不授权边界 |
| 1.2 | 2026-06-14 | host-orchestrator | 按 CR-051 增补策略研究生命周期、研究归档治理、硬件冷热分层、项目迁移和 `quant-lab` / `local_backtest` 身份边界 |
| 1.3 | 2026-06-23 | host-orchestrator | CR131 将 CR 命名 design 文档移出默认 surface；source_hld 的 CR051 历史入口改指向 archive。 |
| 1.4 | 2026-06-24 | host-orchestrator | 按 CR138 增补 Runner Control Plane 与 QMT Gateway Service Layer 运营控制面边界；不覆盖 offline runner core、CR020 readonly gateway 或 CR046 双目标策略交付框架。 |
| 1.5 | 2026-06-24 | host-orchestrator | 根据用户 CP3 反馈刷新 CR138 蓝图：长期 HLD 改用功能域命名；Gateway P0 收敛 REST-only；补齐交易日历、佣金 / 费用模型、收益 / PnL 查询；runtime policy 改为按需授权。 |
| 1.6 | 2026-06-26 | host-orchestrator | 根据用户阶段目标重定义，将本文明确为 `quant-lab` 项目蓝图而非端到端策略研究蓝图；归档 v1.5；新增项目级五阶段路线、策略类型适配边界和成熟多因子策略生产 / 模拟盘观察门禁。 |
| 1.7 | 2026-06-27 | codex | 增补统一因子目录与查询 CLI 边界；FEAT-03 拥有 FactorCatalogEntry，FEAT-13 / FEAT-14 和 mature runner 只能消费已登记因子，不隐式新增因子。 |
| 1.8 | 2026-06-27 | codex | 增补高级多因子模型评估门禁；FEAT-03 拥有 FactorModelValidationReport，mature admission 必须消费 GRS、因子溢价、经济显著性、时间切分、测试资产多样性、样本外、壳价值、做空可行性、政策周期和稳健性评估结果。 |
| 1.9 | 2026-06-27 | codex | 收紧 Stage 3 阶段目标：Stage 3 必须输出评估通过的成熟多因子策略；blocked / insufficient_data 运行只能作为诊断基线，不得进入 Stage 4。 |
| 1.10 | 2026-06-27 | codex | 增补异象发现与异象研究闭环；FEAT-03 拥有 AnomalyCandidate、AnomalyResearchReport 和 AnomalyAdmissionDecision，异象必须先完成理论驱动、Harvey 标准、单调性、控制因子、时间切分、A 股可行性和经济逻辑审查后才能升级为因子候选。 |
| 1.11 | 2026-06-27 | codex | 增补自动异象发现系统；FEAT-03 通过受控模板生成候选、记录搜索空间、执行多重检验控制，并仅将通过准入的候选动态接入因子目录和 Stage 3 候选搜索。 |
| 1.12 | 2026-06-28 | codex | 增补研究引擎稳定模块边界；FEAT-03 研究实现入口改用领域名模块，旧 chapter/stage/root 脚本只作兼容或归档，不再作为主实现面。 |
| 1.13 | 2026-06-28 | meta-se | 按 CR-139「Strategy Data Foundation」parent CP3 增补"策略生产数据底座"能力地图增量：FEAT-02 写侧/读侧分层、ML feature/label/artifact 层归属（FEAT-03）、读侧语义（PIT reader/panel reader/dedup/published selector/读审计）、run evidence + broker facts + 交易审计链 run-id 贯通、配置类事实源层（benchmark/commission/universe·risk policy/政策周期）版本化 + release 闭环；补跨 Feature 流程 FLOW-20..FLOW-23（reader→published pointer、ML→panel reader、broker facts→run-id 贯通、配置类事实源→release 闭环）；companion HLD「Strategy Data Foundation」收口 HLD-DATA-LAKE 范围（superseded-in-scope）。AGA-1/AGA-3/AGA-5 推荐方案 CP3 已确认 A1/C1/E1（2026-06-28T17:30:00+08:00）。 |
| 1.14 | 2026-06-28 | host-orchestrator | CP3 approved，AGA-1/3/5 确认 A1/C1/E1，pending-cp3 → confirmed-cp3。 |

## 蓝图定位

| 项 | 当前口径 |
|---|---|
| 项目类型 | 本地 Python 量化研究、回测、数据湖、策略交付合同和交易接入准备工具 |
| 当前执行状态 | `process/STATE.md` 中 CR-046 保持 paused at CP6 / ready-for-verification；CR-051 已通过 CP3，正在进行 strategy research lifecycle / quant-lab migration 的 CP4 story-planning |
| 蓝图作用 | 为长期维护提供能力边界和数据归属索引，不替代 HLD、ADR、LLD 或检查点 |
| 授权边界 | 本文不授权真实 provider、lake、publish、gateway、QMT、MiniQMT、simulation、live、凭据读取、账户操作、NAS 扫描 / 挂载 / 搬迁、目录重命名、远端仓库改名、git push 或历史文件批量替换 |

### 项目级阶段路线

| 阶段 | 名称 | 目标 | 数据湖边界 | 退出条件 | 后续衔接 |
|---|---|---|---|---|---|
| Stage 1 | `quant-lab` 项目蓝图刷新 | 在现有蓝图基础上合并项目级能力边界、策略类型适配和五阶段路线；归档旧蓝图副本 | 不连接数据湖 | `BLUEPRINT` / `DOMAIN-MAP` / `DEPENDENCY-MAP` 已明确多因子、事件型、机器学习和规则策略的统一接入边界 | Stage 2 |
| Stage 2 | 多因子研究框架升级 | 在蓝图基础上升级 `quant-lab` 中的多因子研究框架，使其支撑 Stage 3 策略生产 | 不连接数据湖；只能使用 fixture、样例输入、schema、静态检查或 typed unavailable | FactorSpec、FactorRunSpec、factor panel、label window、evaluation、portfolio/risk、admission package 和 evidence index 合同齐备 | Stage 3 |
| Stage 3 | 生产评估通过的成熟多因子策略 | 在研究机上连接数据湖，基于真实数据研究、迭代并输出一条可解释、可审计、可扩展到真实股票池且通过高级评估门禁的多因子策略 | 可连接数据湖；必须记录数据 release、lineage、PIT universe、available_at 和版本指针 | `FactorModelValidationReport` 核心门禁无 blocked、mature StrategyAdmissionPackage 为 PASS、runner offline / plan-only / preflight-only 证据通过、no-real-op 计数为 0；blocked / insufficient_data 运行只能作为诊断基线 | Stage 4 |
| Stage 4 | 模拟盘运行与观察 | 运行模拟盘并观察策略表现，通过日常调整达到实盘入口条件 | 只消费 Stage 3 已评估通过并发布的策略包、数据版本和运行证据；runtime 必须逐次授权 | 模拟盘观察周期、收益 / 回撤 / 换手 / 暴露 / 异常 / reconciliation 证据满足 small_live 候选门槛 | Stage 5 |
| Stage 5 | 小额实盘与实盘运行 | 通过独立 live switch CR 进入小额实盘，再按门禁升级实盘 | live 数据、账户、订单和 broker facts 必须独立授权并脱敏 | small_live 通过风险接受、回滚、kill switch、资金限制和人工门禁；scale live 另行审批 | 不自动升级 |

## 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖 Story / CR | Owner Feature |
|---|---|---|---|---|
| CAP-01 | 本地离线研究与轻量日频回测 | 在断网或本地缓存条件下完成策略研究、参数扫描、候选报告和偏差披露 | STORY-001..013、CR-006、CR-008、CR-011、CR-025 | FEAT-01 |
| CAP-02 | 生产级市场数据湖 | 以 raw / manifest / canonical / gold / quality / catalog 形成可审计、可发布、可回滚的 current truth | CR-004、CR-005、CR-007、CR-010、CR-014、CR-017、CR-018 | FEAT-02 |
| CAP-03 | 多因子研究闭环 | 用本项目自有 FactorCatalogEntry / AnomalyCandidate / AnomalyDiscoveryRun / FactorSpec / FactorRunSpec / FactorPanel / LabelWindow / FactorModelValidationReport / report catalog 形成可复跑研究链路 | CR-011、CR-019、CR-030、factor-catalog-cli、factor-model-validation、anomaly-discovery-research、automatic-anomaly-discovery | FEAT-03 |
| CAP-04 | 执行语义对齐与可选后端参考 | 把 lightweight engine、Backtrader optional semantic reference 和 order intent draft 的差异显式化 | CR-025、CR-030 | FEAT-04 |
| CAP-05 | QMT C/S Gateway 与只读运行准入 | 通过 Linux C 侧 client / CLI 和 Windows S 侧 gateway 打通受控只读查询准备 | CR-019、CR-020 | FEAT-05 |
| CAP-06 | OMS、风控、Broker Lake 与阶段激活 | 为后续 simulation / live_readonly / small_live / scale_up 提供订单状态、风控、对账和 kill switch 边界 | CR-015、CR-016、CR-017、CR-021..024 candidate | FEAT-06 |
| CAP-07 | 安全、授权与 no-real-operation 治理 | 保证 CP、Story verified、runbook 和 gateway health 不被误读为真实操作授权 | 全部 CR，重点 CR-014、CR-019、CR-020、CR-025、CR-030 | FEAT-07 |
| CAP-08 | 文档、Runbook 与发布证据 | 让用户能按当前安全边界操作、验证和恢复，同时保留审计证据 | README、docs/USER-MANUAL.md、docs/QMT-*.md、CP8 | FEAT-08 |
| CAP-09 | 双目标策略交付框架 | 让同一研究策略核心可按合同交付到 QMT 终端策略包和 MiniQMT runner 包，并由统一验证框架审查 | CR-046、CR047-candidate、CR049-candidate、CR051-candidate | FEAT-09 |
| CAP-10 | 策略研究生命周期与项目迁移治理 | 将 idea、资料、研究项目、运行证据、归档、策略交付候选和 quant-lab 迁移路径纳入统一生命周期与安全边界 | CR-051、CR052..CR056-candidate | FEAT-10 |
| CAP-13 | 项目级策略类型适配框架 | 让多因子、事件型、机器学习和规则型策略都能通过统一研究、回测、准入、runner 和交付证据链路接入 `quant-lab` | Stage 1、Stage 2、后续事件 / ML CR | FEAT-13 |
| CAP-14 | 成熟策略生产与模拟盘观察门禁 | 将成熟策略从研究证据升级为评估通过、runner 可消费的准入包，再进入长期 simulation observation 和 small_live 候选判断 | Stage 3、Stage 4、Stage 5 | FEAT-14 |

## Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 / 对象 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-01 | 本地研究与轻量回测核心 | `engine/`、`strategies/`、回测、扫描、候选、研究报告 metadata、只读 ResearchDataset 消费 | provider 抓取、真实 lake 写入、catalog publish、QMT native 调用、真实交易 | BacktestRun、PortfolioResult、MetricsReport、CandidateReport、BiasAudit | published catalog、quality/readiness、benchmark、factor panel、order intent draft 输入 | `market_data.connectors`、`market_data.runtime`、provider SDK、QMT / XtQuant、credential env |
| FEAT-02 | 生产级市场数据湖 | `market_data/` 生产链路、lake layout、dataset schema、run / manifest、normalization、validation、quality/readiness、catalog publish、replay、retention、rollback | 策略逻辑、因子评价算法、OMS、QMT 操作、broker lake | MarketDataRun、Manifest、DatasetCandidate、CatalogCurrentTruth、QualityReport、ReadinessReport、ClaimBoundary | CR / HLD / dataset requirements、source registry | `engine` 反向写入、trading、broker lake、QMT gateway |
| FEAT-03 | 研究数据集与多因子研究闭环 | `research_input_v1`、FactorCatalogEntry、AnomalyCandidate、AnomalyDiscoveryRun、AnomalyResearchReport、AnomalyAdmissionDecision、FactorSpec、FactorRunSpec、FactorPanel、LabelWindow、FactorEvaluationReport、FactorModelValidationReport、MultiFactorCombiner、ExperimentManifest、ResearchReportCatalog、StrategyAdmissionPackage、因子查询 CLI | 数据事实源写入、外部框架默认运行、真实 QMT / simulation / live、catalog publish、mature runner-local 计算迁移、政策周期事实源生产、无先验逻辑的数据挖掘自动准入、黑箱公式搜索 | FactorCatalogEntry、FactorAvailabilityStatus、FactorUsageRef、AnomalyCandidate、AnomalyDiscoveryRun、AnomalyResearchReport、AnomalyAdmissionDecision、FactorSpec、FactorRunSpec、FactorPanelContract、LabelWindowSpec、FactorModelValidationReport、ValidationGateDecision、ExperimentManifest、StrategyAdmissionPackage | FEAT-02 published data / policy cycle dataset candidate、FEAT-01 lightweight results、FEAT-04 semantic diff | provider、lake write、external project runtime、QMT gateway direct operation、未登记因子的隐式消费、伪造政策周期或 broker shortability facts、无经济逻辑异象进入因子目录、未记录搜索空间的自动发现结果进入 Stage 3 |
| FEAT-04 | 执行语义对齐与可选后端参考 | Backtrader optional semantic reference、clean feed gate、semantic diff schema、target portfolio / `order_intent_draft_v1` 衔接、license / no-copy guardrail | 默认回测主路径替代、源码迁移、真实 broker store、QMT live store、provider / lake / publish | SemanticDiffReport、BackendAvailability、OrderIntentDraft | clean feed、lightweight run output、data quality metadata | default dependency、source copy、real broker/QMT/provider |
| FEAT-05 | QMT C/S Gateway 与只读运行准入 | Linux C 侧 REST client / Typer CLI、Windows S 侧 gateway lifecycle、pairing/HMAC、allowlist/scope/nonce、QMT login/session ready、`query_positions` readonly endpoint | OMS 下单、simulation/live、账户写入、发单/撤单、broker lake 写入、策略准入判断 | GatewayConfig、PairingToken、HMACScope、QMTLoginSession、QueryPositionsResult | authorization record、redacted `.env` refs、endpoint matrix、stage gate status | C 侧导入 xtquant、gateway 默认公网暴露、无授权真实 QMT 调用 |
| FEAT-06 | OMS / 风控 / Broker Lake / 阶段激活 | target portfolio 到 order intent、OMS 状态机、pre-trade hard risk、broker lake schema、stage gate、reconciliation、kill switch、incident playbook | 因子生成、市场数据湖写入、gateway lifecycle、凭据存储 | OrderIntent、OMSOrder、BrokerEvent、BrokerLakeRecord、StageGate、AuthorizationRecord、KillSwitchEvent | FEAT-03 admission package、FEAT-05 gateway capabilities、FEAT-02 raw price / metadata | strategies 直连、复权价执行、无授权 adapter 调用 |
| FEAT-07 | 安全、授权与 no-real-operation 治理 | runtime authorization、risk acceptance、credential redaction、no-real-operation counters、forbidden import、human gate 不授权项 | 业务算法或数据 schema 本身 | AuthorizationDecision、NoRealOpCounter、RedactionPolicy、ForbiddenDependencyRule | 全部 Feature 的设计 / 测试 / 日志证据 | 任何通过文档或健康检查隐式升级运行授权的路径 |
| FEAT-08 | 文档、Runbook 与发布证据 | README、USER-MANUAL、QMT 安装 / 运行手册、incident playbook、release readiness、CP8 用户终验摘要 | 改变业务范围、授权真实操作、修改事实源 | Runbook、UserManualSection、ReleaseNote、RollbackPlan、FeedbackEntry | HLD / ADR / verification report / CP8 | 文档声明超出已验证状态或绕过 gate |
| FEAT-09 | QMT / MiniQMT 双目标策略交付框架 | 策略核心合同、策略包目录、QMT terminal target、MiniQMT runner target 安装设计、统一验证证据模型、后续策略交付门禁 | 具体策略交付、QMT 终端运行验证、MiniQMT 真实连接、真实安装、submit/cancel、simulation/live | StrategyCoreContract、StrategyPackageContract、QMTTerminalTargetContract、MiniQMTRunnerTargetContract、StrategyValidationEvidence | FEAT-03 StrategyAdmissionPackage、FEAT-04 OrderIntentDraft、FEAT-06 风控边界、FEAT-07 授权记录 | 策略核心直连 QMT / XtQuant、验证框架触发真实运行、安装设计读取凭据或写真实运行目录 |
| FEAT-10 | 策略研究生命周期与项目迁移治理 | InformationSource、StrategyIdea、ResearchProject、ResearchProtocol、ResearchRun、ValidationEvidence、ResearchArchiveManifest、ProjectIdentity、MigrationInventory、Follow-up CR roadmap | 具体策略实现、真实交易、provider/lake/publish、QMT/MiniQMT runtime、目录实际重命名、NAS 实际扫描 / 挂载 / 搬迁、远端仓库改名或 git push | InformationSource、StrategyIdea、ResearchProject、ResearchProtocol、ResearchRun、ValidationEvidence、ResearchArchiveManifest、StrategyTaxonomyEntry、ProjectIdentity、MigrationInventory | FEAT-02 catalog/data release、FEAT-03 admission package、FEAT-09 StrategyCoreContract / StrategyValidationEvidence | provider SDK、QMT / XtQuant / MiniQMT、broker lake write、真实 NAS 操作、凭据读取、git push / remote rename、批量重写历史 process 证据 |
| FEAT-13 | 策略类型适配与统一研究合同 | StrategyTypeAdapter、SignalSet、StrategyCandidate、ResearchEvidenceIndex、StrategyTypeTaxonomy；负责把多因子、事件型、机器学习和规则型策略归一到统一研究 / 回测 / 准入合同，并只引用 FEAT-03 已登记因子 | 替代 FEAT-03 的多因子算法实现、替代 FEAT-02 数据事实源、直接驱动 gateway / OMS / live、把某一种策略类型写死为全局唯一模型、私有新增因子目录 | StrategyTypeAdapter、SignalSet、StrategyCandidate、ResearchEvidenceIndex、StrategyTypeTaxonomy | FEAT-02 current truth、FEAT-03 factor catalog / factor outputs、FEAT-10 research lifecycle、FEAT-14 admission / observation gate | provider/lake write、QMT / XtQuant、broker lake、live runtime、策略核心直连交易系统、未登记因子 |
| FEAT-14 | 成熟策略生产、准入和模拟盘观察门禁 | MatureStrategyDefinition、UniversePolicy、PortfolioRiskPolicy、SimulationObservationPlan、ReadinessDecision；负责 Stage 3 到 Stage 5 的门禁、证据和风险接受边界，并校验成熟策略引用的因子已在 FEAT-03 目录登记、FactorModelValidationReport 核心门禁已通过且 mature admission 为 PASS | 具体因子计算实现、数据湖生产、gateway lifecycle、真实下单、small_live/live 授权本身、隐式新增 mature runner 因子、替代 FEAT-03 统计评估、把 blocked 研究运行包装成 Stage 4 输入 | MatureStrategyDefinition、UniversePolicy、PortfolioRiskPolicy、SimulationObservationPlan、ReadinessDecision | FEAT-03 factor catalog / FactorModelValidationReport / research evidence、FEAT-13 research evidence、FEAT-11 runner evidence、FEAT-12 gateway health、FEAT-07 authorization | 直接读取 raw account / raw order / credential、绕过 runner P0-P4、把 simulation readiness 自动升级为 live readiness、未登记因子进入策略包、跳过高级评估门禁、blocked 策略进入模拟盘观察 |

## 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-01 | 用户显式授权数据生产 | FEAT-02 -> FEAT-01 / FEAT-03 | FEAT-02 | plan / run / normalize / validate 任一失败时不得 publish；consumer 返回 `required_missing` / `blocked_claims` | `tests/test_cr014_*`、`tests/test_cr018_*` |
| FLOW-02 | 研究运行需要生产数据 | FEAT-01 -> FEAT-02 | FEAT-02 | FEAT-01 只读 catalog，不自动补数；缺口返回 structured unavailable | `tests/test_cr010_consumer_boundary.py`、`tests/test_cr014_research_consumer_boundary.py` |
| FLOW-03 | 多因子研究闭环 | FEAT-03 -> FEAT-01 / FEAT-04 | FEAT-03 | factor / label / report 任一 gate 失败时 admission package blocked；异象候选未完成先验逻辑、Harvey t>=3、单调性、控制因子、时间切分、成本、经济逻辑和多重检验审查时不得升级为可计算因子；自动发现只能从受控模板生成候选并记录 candidate_count；研究引擎主实现必须使用领域名模块 | `tests/test_cr030_*`、`tests/test_chapter5_anomalies.py`、`tests/test_anomaly_discovery.py`、`tests/test_script_entrypoint_naming.py` |
| FLOW-04 | 研究输出转执行草稿 | FEAT-03 / FEAT-04 -> FEAT-06 | FEAT-04 / FEAT-06 | `order_intent_draft_v1` 不等于真实 order；缺 execution raw policy 时 blocked | `tests/test_cr025_order_intent_draft_contract.py`、`tests/test_cr030_strategy_admission_package.py` |
| FLOW-05 | QMT 只读实机验证 | FEAT-05 -> FEAT-07 -> FEAT-08 | FEAT-05 | 缺 MiniQMT 权限、凭据、pairing、scope 或 session ready 时 blocked，不关闭 CR-020 | `tests/test_cr020_*` + 用户脱敏 evidence |
| FLOW-06 | 后续 simulation / live 路线 | FEAT-03 -> FEAT-06 -> FEAT-05 -> FEAT-07 | FEAT-06 | CR-021..024 未启动 / 未授权时全部 blocked；gateway capabilities 不升级授权 | `tests/test_cr016_*`、后续 CR |
| FLOW-07 | 文档发布和终验 | FEAT-08 -> FEAT-07 | FEAT-08 | README / runbook / CP8 不得声明未授权真实能力 | docs guardrail tests、CP8 |
| FLOW-08 | 研究策略进入双目标交付 | FEAT-03 / FEAT-04 -> FEAT-09 -> FEAT-07 | FEAT-09 | 缺少策略核心合同、order intent draft、风险假设或验证证据时，策略包不得进入 CR047 交付 | CR046 CP3/CP5、后续 CR047 |
| FLOW-09 | MiniQMT runner 安装设计审查 | FEAT-09 -> FEAT-07 -> FEAT-08 | FEAT-09 | 本 CR 只允许 install dry-run 设计；真实安装、连接和运行必须进入 CR049 / runtime authorization gate | CR046 验证框架、后续 CR049 |
| FLOW-10 | 策略想法进入研究闭环 | FEAT-10 -> FEAT-03 -> FEAT-09 -> FEAT-07 | FEAT-10 / FEAT-03 / FEAT-09 | idea、protocol、run、validation 任一证据缺失时只能保持 research-only 或 blocked，不得升级为 delivery_candidate | CR051 CP5/CP7、后续 CR052 |
| FLOW-11 | 项目身份和仓库结构迁移 | FEAT-10 -> FEAT-08 -> FEAT-07 | FEAT-10 | 仅在 inventory、Git archive、mechanical move plan、legacy alias 验证和用户授权齐备后才允许真实迁移；CP4/CP5 只做设计 | CR051 CP4/CP5、后续迁移授权门禁 |
| FLOW-17 | 策略类型研究结果归一化 | FEAT-03 / 事件策略后续 Feature / ML 后续 Feature -> FEAT-13 -> FEAT-14 | FEAT-13 | 缺少 StrategyTypeAdapter、SignalSet、StrategyCandidate 或 ResearchEvidenceIndex 时不得进入成熟策略准入 | Stage 1 / Stage 2 design checks |
| FLOW-18 | 多因子框架升级到策略生产准备 | FEAT-03 -> FEAT-13 -> FEAT-14 | FEAT-03 / FEAT-13 | Stage 2 不连接数据湖；真实数据缺口只能 typed unavailable，不得伪造 lineage、PIT universe、policy cycle coverage 或 shortability facts | Stage 2 framework tests |
| FLOW-19 | 成熟多因子策略进入模拟盘观察 | FEAT-02 -> FEAT-03 -> FEAT-13 -> FEAT-14 -> FEAT-11 / FEAT-12 -> FEAT-07 | FEAT-14 / FEAT-11 | Stage 3 必须产出评估通过的策略；research evidence、FactorModelValidationReport 或 mature admission 缺失 / blocked 时只能记录为失败候选，不得进入 Stage 4；核心门禁含因子溢价、经济显著性、样本外和数据偏差审计；Stage 4 runtime 必须逐次授权并从 P0 开始 | Stage 3 / Stage 4 readiness checks |

## 共享能力

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-01 | Claim Boundary | FEAT-01、FEAT-03、FEAT-08 | FEAT-02 / FEAT-07 | data lake -> consumer / docs | 缺口进入 `blocked_claims`，不得 silent downgrade |
| SH-02 | No-real-operation Counters | 全部 Feature | FEAT-07 | runtime / tests / docs -> safety evidence | 计数非 0 且无授权时 blocked |
| SH-03 | Order Intent Draft | FEAT-03、FEAT-04、FEAT-06 | FEAT-04 | research -> trading governance | 只能 draft，不能触发 adapter |
| SH-04 | Runtime Authorization Record | FEAT-05、FEAT-06、FEAT-07 | FEAT-07 | human gate / per-run -> runtime | 缺授权时 fail-closed |
| SH-05 | Current Truth Catalog | FEAT-01、FEAT-03、FEAT-04 | FEAT-02 | data lake -> readers | 未 publish 返回 unavailable |
| SH-06 | Runbook Boundary Statements | 用户、FEAT-05、FEAT-06 | FEAT-08 / FEAT-07 | docs -> operator | 文档必须列不授权项 |
| SH-07 | Strategy Core Contract | FEAT-03、FEAT-04、FEAT-09 | FEAT-09 | research/admission -> strategy package | 合同缺字段时 blocked，不生成目标包 |
| SH-08 | Strategy Validation Evidence | FEAT-07、FEAT-08、FEAT-09 | FEAT-09 / FEAT-07 | package validation -> CP gate / docs | 只有 fixture/static/dry-run 证据，不得声称 terminal/runtime verified |
| SH-09 | Research Archive Manifest / Run Manifest | FEAT-01、FEAT-02、FEAT-03、FEAT-08、FEAT-10 | FEAT-10 | research run -> archive manifest / docs | 缺 commit、data release、config hash、seed 或 artifact ref 时 blocked |
| SH-10 | Project Identity Alias / Migration Guardrail | 全部 Feature | FEAT-10 / FEAT-07 | project docs / migration plan -> consumers | `quant-lab` 为 canonical；`local_backtest` 作为 legacy alias，不批量重写历史审计 |
| SH-15 | Strategy Type Adapter Contract | FEAT-03、FEAT-10、FEAT-13、FEAT-14 | FEAT-13 | strategy-specific research output -> SignalSet / StrategyCandidate | adapter 缺字段时 blocked，不进入成熟策略准入 |
| SH-16 | Mature Strategy Admission Gate | FEAT-03、FEAT-11、FEAT-13、FEAT-14、FEAT-07 | FEAT-14 / FEAT-07 | research evidence -> FactorModelValidationReport -> mature StrategyAdmissionPackage -> runner preflight | 缺 FactorSpec、UniversePolicy、lineage、risk、evidence index 或评估通过状态时 blocked |
| SH-17 | Simulation Observation Evidence | FEAT-11、FEAT-12、FEAT-14、FEAT-08 | FEAT-14 / FEAT-11 | simulation run -> observation report -> small_live candidate decision | 未完成观察周期或异常未闭环时不得升级 small_live |
| SH-18 | Unified Factor Catalog | FEAT-03、FEAT-13、FEAT-14、FEAT-11、FEAT-08 | FEAT-03 | factor registry -> mature runner FactorSpec / factor research modules / CLI query | 未登记因子不得被 mature runner、factor research modules 或成熟策略准入静默消费；缺计算器必须声明 runner-local、proxy-only 或 blocked |
| SH-19 | Advanced Factor Model Validation | FEAT-03、FEAT-13、FEAT-14、FEAT-10、FEAT-08 | FEAT-03 | factor panel / labels / factor returns / universe / tradability / policy cycle config -> FactorModelValidationReport -> mature admission | 核心门禁 blocked 时 mature admission blocked；Stage 3 不得出阶段；非核心缺口必须进入 risk_warnings 或 unavailable，不得伪造 GRS、做空可行性、政策周期或壳价值控制证据 |
| SH-20 | Anomaly Discovery / Research Gate | FEAT-03、FEAT-13、FEAT-14、FEAT-10、FEAT-08 | FEAT-03 | theory / behavior / microstructure / financial-extension hypothesis -> AnomalyCandidate -> AnomalyResearchReport -> AnomalyAdmissionDecision -> FactorCatalogEntry candidate | 无先验逻辑、Harvey t<3、分组非单调、控制因子 alpha 不显著、时间切分失败、扣费后不可行、壳价值 / 做空 / 政策周期缺口未披露或经济故事为空时不得升级为 Stage 3 因子 |
| SH-21 | Automatic Anomaly Discovery | FEAT-03、FEAT-13、FEAT-14 | FEAT-03 | controlled templates + feature panel + model returns -> AnomalyDiscoveryRun -> multiple-testing controlled admission -> dynamic FactorCatalogEntry extras -> Stage 3 candidate FactorSpec | 不允许黑箱公式搜索、不自动 publish catalog、不把一次搜索胜者硬编码为静态因子；未通过多重检验或未记录搜索空间的候选不得进入 Stage 3 搜索 |
| SH-22 | Research Engine Stable Modules | FEAT-03、FEAT-13、FEAT-14、FEAT-08 | FEAT-03 | stable engine modules / stable scripts -> tests / docs / research runners | `engine/factor_replication.py`、`research_data_readiness.py`、`factor_model_research.py`、`anomaly_research.py`、`factor_robustness.py`、`factor_portfolio_practice.py` 是长期主入口；旧 `engine/chapter*` 已归档到 `docs/legacy/archive/engine/`，旧 root 脚本实现必须归档到 `scripts/legacy/*` |

## 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-BP-001 | follow_up_tracking | 是否把 CR-031 的蓝图索引作为后续 CR-021..024 的默认设计入口 | 推荐：作为默认入口，但不替代 legacy HLD/ADR 审计 | 仅保留为临时说明 | 推荐方案能降低后续翻阅成本；备选不改变现状但边界继续分散 | 影响后续 QMT simulation/live 设计交接 | 若发现索引与 legacy 事实冲突，以 legacy HLD/ADR/Story 为准并修订蓝图 |
| DQ-BP-002 | implementation | 是否立即回写 134 个 Story 的 `feature_design_refs` | 推荐：本轮不批量回写，只在后续变更 Story 中引用矩阵 | 批量回写所有 Story | 推荐方案低风险；批量回写会污染已验证证据且容易引入误差 | 影响 Story 历史审计和 git diff 规模 | 后续新 CR 或重开 Story 时增量回写 |
| DQ-BP-CR046-01 | architecture | CR046 是否新增独立 FEAT-09 承载双目标策略交付框架，而不把能力并入 FEAT-05 / FEAT-06 | 推荐：新增 FEAT-09，FEAT-05/06 仍保留 gateway 和交易治理边界 | 并入 FEAT-05；并入 FEAT-06 | 推荐方案避免把“策略交付合同”误读为 gateway runtime 或 OMS 授权；并入方案更少 Feature 但边界混淆 | 影响 CR047 策略包和 CR049 runner install 的消费入口 | 若后续只保留 QMT terminal 且放弃 MiniQMT，可将 FEAT-09 降级为 FEAT-05/06 的子能力 |
| DQ-BP-CR051-01 | architecture | CR051 是否新增 FEAT-10 承载策略研究生命周期和 quant-lab 迁移治理 | 推荐：新增 FEAT-10，FEAT-03 继续负责因子研究和 admission，FEAT-09 继续负责策略交付包 | 并入 FEAT-03；并入 FEAT-08 | 推荐方案能把研究 lifecycle、archive、项目身份和迁移边界集中管理；并入 FEAT-03 会扩大研究算法边界，并入 FEAT-08 会把迁移治理误降级为文档刷新 | 影响 CR052..CR056 的进入门禁和本项目迁移顺序 | 若 CR051 CP5 发现 FEAT-10 只剩文档更新，可降级为 FEAT-08 technical-note；若真实迁移启动，仍需独立 runtime_authorization / migration gate |
| DQ-BP-20260626-01 | architecture | 当前蓝图是否改为 `quant-lab` 项目蓝图，而不是新增一份端到端策略研究蓝图 | 推荐：在现有 `docs/design/BLUEPRINT.md` 上合并项目级阶段路线和策略类型边界，并归档 v1.5 | 新增独立 strategy lifecycle blueprint；只更新 runner 阶段文档 | 推荐方案避免蓝图分叉，保留 `quant-lab` 项目级唯一入口；独立文档容易变成旁路设计 | 影响后续 Stage 1 恢复入口、Stage 2 框架升级和 Stage 3 策略生产 | 若蓝图膨胀难维护，可在后续 CR 中拆出子文档，但 `BLUEPRINT.md` 仍保留唯一索引 |
| DQ-BP-20260626-02 | implementation | Stage 2 多因子研究框架升级是否连接数据湖 | 推荐：不连接数据湖，只升级合同、schema、框架、fixture 和测试；真实数据生产留到 Stage 3 | Stage 2 即连接数据湖；Stage 2 只写文档不改框架 | 推荐方案降低数据环境耦合并为研究机实施留接口；直接连湖会扩大运行授权和环境风险；只写文档不能支撑策略生产 | 影响 Stage 2 验证边界和 Stage 3 数据湖接入准备 | 当 Stage 2 合同验证通过且用户转到研究机后，进入 Stage 3 数据湖实施 |
| DQ-BP-20260626-03 | architecture | 多因子、事件型、机器学习和规则型策略是否统一输出 `SignalSet` / `StrategyCandidate` / `StrategyAdmissionPackage` | 推荐：统一输出合同，策略类型差异留在 adapter 内部 | 每种策略单独 runner 输入；只支持多因子 | 推荐方案支持扩展和统一测试；单独输入会导致 runner、风控和测试矩阵分叉；只支持多因子不满足项目目标 | 影响后续事件型、ML 策略 CR 和测试交付框架 | 若某类策略需要特殊 execution primitive，可扩展 adapter 字段，但不得绕过 mature admission gate |

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 每个 Feature / Epic 有职责、非职责和数据归属 | PASS | §Feature / Epic 边界 |
| 跨 Feature 流程写明 Owner 和失败路径 | PASS | §跨 Feature 流程 |
| 共享能力写明调用方向和降级策略 | PASS | §共享能力 |
| 运行授权边界未被蓝图放大 | PASS | §蓝图定位、FEAT-07、FEAT-09、FEAT-10、FEAT-13、FEAT-14、DQ-BP-CR046-01、DQ-BP-CR051-01、DQ-BP-20260626-02 |

## CR138 增量：Runner / QMT Gateway Operational Control Plane

> 本节是 `process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` 的蓝图索引。它只冻结运营控制面设计；CP3 approve 不自动授权 runtime、QMT / MiniQMT / XtQuant、凭据、账户、行情、订单读取、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。后续验证如必要，必须按动作范围走独立 `runtime_authorization` gate。

### 能力地图增量

| Capability ID | 能力域 | 用户价值 | 覆盖 Story / CR | Owner Feature |
|---|---|---|---|---|
| CAP-11 | Runner Control Plane | 让运营负责人能生成运行计划、做盘前确认、观察执行、接入事件/信号、再平衡、查证据、复盘、恢复和变更策略生命周期 | CR-138、UC-33..UC-43 | FEAT-11 |
| CAP-12 | QMT Gateway Service Layer | 让 QMT / MiniQMT / XtQuant 触达被封装为有 health、capabilities、会话、查询、订阅、回报、恢复、审计和配置变更的服务层 | CR-138、UC-44..UC-50 | FEAT-12 |

### Feature / Epic 边界增量

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 / 对象 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-11 | Runner Control Plane | RunPlan、PreflightResult、RunnerCommand、RunState、RunEvidence、ReviewSummary、IncidentRecord、StrategyChangePlan；负责 UC-33..UC-43 的用户运营路径 | QMT / MiniQMT / XtQuant native 调用、真实 account / quote / order query、submit/cancel、gateway lifecycle、策略核心合同定义 | RunPlan、RunnerCommand、RunState、RunEvidence、ReviewSummary、IncidentRecord、StrategyChangePlan | FEAT-03 StrategyAdmissionPackage、FEAT-04 OrderIntentDraft、FEAT-09 StrategyPackageContract、FEAT-12 GatewayHealth / ExecutionReport | xtquant / QMT native API、credential env、broker lake direct write、provider/lake/catalog write |
| FEAT-12 | QMT Gateway Service Layer | GatewayConfig、GatewayHealth、CapabilitySnapshot、TradingSession、TradingCalendar、ReadonlyQueryResult、CommissionSchedule、PnLSnapshot、MarketSubscription、GatewayCommand、ExecutionReport、GatewayAuditRecord、GatewayChangePlan；负责 UC-44..UC-50 的服务层路径 | 策略信号生成、目标组合计算、策略版本发布决策、研究准入判断、offline runner bundle / registry authority | GatewayConfig、GatewayHealth、CapabilitySnapshot、TradingSession、TradingCalendar、CommissionSchedule、PnLSnapshot、MarketSubscription、ExecutionReport、GatewayAuditRecord、GatewayChangePlan | AuthorizationRecord、OrderIntent、risk decision、RunPlan summary、本地交易日历参考数据 | 策略逻辑、research data lake write、strategy package mutation、未授权 QMT / account / quote / order / submit/cancel |

### 跨 Feature 流程增量

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-12 | 盘前运行计划和确认 | FEAT-09 / FEAT-03 / FEAT-02 -> FEAT-11 -> FEAT-07 / FEAT-12 | FEAT-11 | admission、data release、authorization 或 GatewayHealth 任一缺失时 blocked / manual_review | CR138 HLD §7 SIM-138-01 |
| FLOW-13 | 事件 / 信号到订单意图 | FEAT-11 -> FEAT-04 / FEAT-06 -> FEAT-07 -> FEAT-12 | FEAT-11 / FEAT-06 | 重复信号 idempotent skip；risk fail 或 no auth 时 adapter_calls=0 | CR138 HLD §7 SIM-138-02 |
| FLOW-14 | 行情订阅与恢复 | FEAT-11 -> FEAT-12 -> FEAT-07 | FEAT-12 | no market authorization 时 blocked；xtdata 异常时 degraded / unavailable；P0 通过 REST 管理和拉取事件 | HLD §7 SIM-138-03 |
| FLOW-15 | 未确认订单 / Gateway 故障人工接管 | FEAT-12 -> FEAT-06 -> FEAT-11 -> FEAT-08 | FEAT-11 / FEAT-12 | unknown order 不自动成功、不自动重发；kill switch 不自动解除 | HLD §7 SIM-138-04 |
| FLOW-16 | 交易日历 / 佣金 / 收益查询 | FEAT-11 -> FEAT-12 -> FEAT-07 / FEAT-06 | FEAT-12 / FEAT-06 | 交易日历本地参考缺失时 unavailable；账户佣金 / 收益无授权时 blocked；QMT 不支持时返回 unavailable_with_reason 或 estimated | HLD §11 FLOW-G2 |

### 共享能力增量

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-11 | Runner / Gateway Audit ID | FEAT-11、FEAT-12、FEAT-06、FEAT-07、FEAT-08 | FEAT-12 / FEAT-07 | request -> command -> execution report -> review | 缺 audit_id 时 blocked，不输出可审计 pass |
| SH-12 | GatewayHealth / CapabilitySnapshot | FEAT-11、FEAT-12、FEAT-07 | FEAT-12 | gateway -> runner preflight / ops view | health pass 不升级 runtime 授权；缺授权返回 blocked |
| SH-13 | RunnerCommand / GatewayEvent Contract | FEAT-11、FEAT-12、FEAT-06 | FEAT-11 / FEAT-12 | runner intent -> gateway event stream | schema mismatch fail-closed |
| SH-14 | Reference / Account Query Contract | FEAT-11、FEAT-12、FEAT-06、FEAT-07 | FEAT-12 | runner preflight / review -> calendar / commission / pnl query -> redacted summary | 缺账户授权时 blocked；QMT 不支持佣金 / 收益时返回 unavailable_with_reason，不伪造 broker facts |

### 待确认边界增量

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-BP-CR138-01 | architecture | 是否新增 FEAT-11 / FEAT-12，而不是扩展 FEAT-05/06/09 或 offline runner core | 推荐新增 FEAT-11 Runner Control Plane 与 FEAT-12 Gateway Service Layer | 并入 FEAT-05；并入 FEAT-09；扩展 strategy-runner-core | 推荐方案边界清晰；备选文件少但混淆只读准入、策略包和运营 runtime | 影响 CP4 Story 分组和后续 LLD owner | 若用户将 CR138 缩小为 Runner-only 或 Gateway-only，可降级为单 Feature 增量 |
| DQ-BP-CR138-02 | runtime_authorization | CP3 approval 是否自动授权 runtime / QMT / account / quote / order / trading，以及后续是否可按需授权验证 | 推荐 CP3 不自动授权；后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权 | blanket deny；CP3 直接授权只读 / 行情 / 交易 | 推荐方案兼顾安全和验证可行性；blanket deny 会阻断必要验证，CP3 直接授权风险过高 | 防止 CP3 被误解为运行许可，同时保留后续验证路径 | 用户明确授权运行窗口和边界后发起 runtime_authorization gate |
| DQ-BP-CR138-03 | architecture | 交易日历、佣金 / 费用模型、收益 / PnL 查询是否显式归入 FEAT-12 Query Service | 推荐归入 FEAT-12，并按本地参考数据 / 账户只读授权分层 | 放在 FEAT-11；延后到 OMS/Risk | 推荐方案让 preflight、费用估算、复盘有统一入口；放 Runner 会诱导 Runner 读取账户，延后会缺核心运营能力 | 影响 Gateway query service 和 Runner preflight/review Story | 若实现阶段 QMT 不支持对应查询，保留 unavailable_with_reason 和 estimated 模式 |

### CR138 蓝图自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 每个新增 Feature 有职责、非职责和数据归属 | PASS | FEAT-11、FEAT-12 边界表 |
| 跨 Feature 流程写明 Owner 和失败路径 | PASS | FLOW-12..FLOW-16 |
| 共享能力写明调用方向和降级策略 | PASS | SH-11..SH-14 |
| 运行授权边界未被放大 | PASS | DQ-BP-CR138-02、HLD §12/§13 |

## CR-139 增量：策略生产数据底座（Strategy Data Foundation）

> 本节是 `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md`（companion HLD，写侧/读侧分离，D2 命名）的蓝图索引。它收口 `process/HLD-DATA-LAKE.md`（CR-018，superseded-in-scope）的范围，把数据湖从"已验证小窗口市场数据链路"推进为支撑多因子 + ML 策略从信息收集→回测→模拟盘→实盘全流程可信/可复现/可审计的**策略生产数据底座**。CP3 approve 不授权 runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移（Wave1 N1 后置到基线冻结之后）/Git remote write（REQ-248）。整改性质为"既有对象与 release/lineage/ML feature/run-id 全链路闭环"，**非整体新建**——仅 L3/E4/T7/T8/X1/X2（d1）为纯新建（REQ-249）。

### 能力地图增量

| Capability ID | 能力域 | 用户价值 | 覆盖 REQ / UC | Owner Feature |
|---|---|---|---|---|
| CAP-15 | 策略生产数据底座（写侧/读侧分离） | 让数据湖支撑四阶段全流程：写侧规范写入/PIT 盖戳/去重/lineage/publish/质量门禁，读侧 PIT as-of/panel join/dedup/published selector/读审计 | REQ-201..245、UC-51..57 | FEAT-02（写侧 + 读侧子域） |
| CAP-16 | ML feature/label/artifact 持久化与审计链 | 让 ML 不再旁路数据湖，feature/label/artifact 可版本化、可复现、训练-实盘一致 | REQ-215/219/221..225、UC-53 | FEAT-03 |
| CAP-17 | run evidence + 交易审计链 run-id 贯通 | 让信号→数据→执行全程审计链同 run-id 贯通，复盘可从异常成交回溯到数据版本/订单/成交/对账 | REQ-233/240..245、UC-55 | FEAT-02/03/06/11 |
| CAP-18 | 配置类事实源版本化 + release 闭环 | 让 benchmark/commission/universe·risk policy/政策周期四类配置事实源像 dataset 一样可版本化、可回溯到 release，回测归因可复现 | REQ-236..239、UC-56 | FEAT-02/12/14/03 |
| CAP-19 | schema 契约冻结 + 跨源时点一致性 | 让 schema 契约在模拟盘前冻结且 reader 兼容回退，复权因子 PIT 正确，跨源交易日历/时区对齐，decision_time 强制阻断 lookahead，universe 按时点构建 PIT 成分链 | REQ-202/203/217/220/226..231/235、UC-57 | FEAT-02/03/14 |

### Feature / 数据归属增量（写侧 / 读侧分离，AGA-1 推荐方案 A1，CP3 已确认）

> FEAT-02 内部按写侧子域 + 读侧子域分层（companion HLD 分写侧章 + 读侧章）。读侧统一 panel reader（R1）owner 在 FEAT-02 读侧子域，FEAT-03 只读消费。若 CP5 出现读写文件所有权冲突频繁，升级为独立 FEAT-02R（AGA-1 备选 A3）。

| Feature ID | 子域 | 增补职责 | 拥有数据 / 对象（增量） | 只读数据 | 禁止依赖（增量） |
|---|---|---|---|---|---|
| FEAT-02 | 写侧 | normalize、PIT 盖戳（available_at）、写入去重（C4）、lineage_checksum 回填（M2）、publish/pointer 前移（L2）、日级增量 append（L1）、events schema 修复（C3）、catalog/manifest 定主（M1）、配置类事实源 release 闭环（F1）、schema 演进规则（V4 写侧） | PublishedRelease、CatalogCurrentPointer、LineageChecksum、IncrementalAppendPlan、ConfigFactRelease | — | reader 反向写 canonical；engine 写 lake |
| FEAT-02 | 读侧 | PIT as-of reader（C1）、统一 panel reader（R1）、读层去重（C2b）、published selector、读审计 log（L3）、readiness 读前门禁（L4）、decision_time lookahead 阻断（X3）、DuckDB 只读 adapter（R3）、列裁剪/谓词下推（R4）、replay（L5） | PITAsOfReader、PanelReader、ReadAuditLog、ReadinessGate、DuckDBReadOnlyAdapter | published current truth | 读未发布数据；reader 写 lake；DuckDB 写持久事实源 |
| FEAT-03 | ML feature 层 | feature/label/artifact 持久化层（V3，lake `features/` 子层带版本）、ExperimentManifest 闭环（E1）、模型 artifact hash（E2）、label 泄漏统一 gate（E3）、离线/在线一致性（E4）、split manifest 冻结（E5）、ML 接入 lake 废除旁路（R2）、policy_cycle 版本化（F4） | FeatureArtifact、FeatureVersionSchema、LabelArtifact、ModelArtifactHash、SplitManifest、PolicyCycleFactSource | FEAT-02 published panel / features 子层 | ML 旁路 `--data-dir`/`load_local_frames`；伪造政策周期/shortability facts |
| FEAT-06 | 交易审计链 | BrokerLakeSchema 闭环实盘写 + 订单/成交/持仓审计链（T4）、run-id 贯通 broker event（T6） | BrokerLakeAuditChain、BrokerEventRunIdLink | FEAT-02 run-id、FEAT-11 RunEvidenceIndex | broker lake 写仓库 data/**；qfq/hfq 进执行价 |
| FEAT-11 | run evidence | RunEvidenceIndex 闭环 lake run-id 贯通（T6）、读审计 log run-id 关联（L3） | RunEvidenceRunIdLink | FEAT-02 读审计 log | — |
| FEAT-12 | 配置层（commission） | CommissionSchedule 版本化费用模型事实源 + release 闭环（F2）、成本前置门禁（T5） | CommissionFactSource、CostGate | FEAT-02 config_facts release | 伪造 broker confirmed commission |
| FEAT-14 | 配置层（universe·risk policy） | PortfolioRiskPolicy 版本化 universe/risk policy 事实源 + release 闭环（F3）、PIT universe 成分链（X4） | UniversePolicyFactSource、RiskPolicyFactSource、PITUniverseConstituentChain | FEAT-02 config_facts release、index_members | 固定快照伪装 PIT universe |

### 跨 Feature 流程增量

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-20 | 读侧消费 published current truth | FEAT-02 读侧 → FEAT-01/03 | FEAT-02 写侧（publish） | 未 publish 返回 unavailable；reader 不得读未发布 | UC-52、C1/R1/V1 tests |
| FLOW-21 | ML 接入可信读路径 | FEAT-03 → FEAT-02 读侧（panel reader） | FEAT-03（features/ 子层） | ML 旁路废除；panel reader published 门禁失败 blocked | UC-53、R2/V3 tests |
| FLOW-22 | 信号→数据→执行 run-id 贯通审计链 | FEAT-02 读侧（读审计）→ FEAT-11（RunEvidenceIndex）→ FEAT-06（broker event） | FEAT-02 读审计 / FEAT-11 / FEAT-06 | run-id 断链标注断点，不伪造贯通 | UC-55、L3/T6 tests |
| FLOW-23 | 配置类事实源 release 闭环 | FEAT-02/12/14/03 → FEAT-02 publish（config_facts） | FEAT-02（config_facts release） | 配置缺版本化字段阻断复现声明 | UC-56、F1-F4 tests |
| FLOW-24 | 模拟盘日级推进 + 读前阻断 | FEAT-02 写侧（L1/L2）→ FEAT-02 读侧（L4 门禁）→ FEAT-14 | FEAT-02 | 读前门禁不通过阻断信号生成 | UC-54、L1/L2/L4 tests |
| FLOW-25 | 模拟盘前 schema 冻结 + 跨源时点对齐 | FEAT-02（V4/X1/X2/X3）+ FEAT-14（X4） | FEAT-02 | schema 变更触发 reader 兼容回退；lookahead/跨源不一致阻断 | UC-57、V4/X1-X4 tests |

### 共享能力增量

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-23 | Published Current Truth Selector | FEAT-01/03/04/14 | FEAT-02 读侧 | reader → published pointer | 未 publish 返回 unavailable，不读 candidate |
| SH-24 | Run-id Lineage Key | FEAT-02/03/06/11 | FEAT-02 写侧（生成） | data run → research → broker event | run-id 断链标注断点，不伪造贯通 |
| SH-25 | Config Fact Release Pointer | FEAT-03/12/14 | FEAT-02（config_facts publish） | config fact → release → consumer | 配置缺版本阻断复现声明 |
| SH-26 | Read-side Safety Gate（PIT/dedup/readiness/lookahead） | FEAT-01/03/14 | FEAT-02 读侧 | reader → gate → panel | 任一门禁失败 blocked，不静默降级 |

### 待确认边界增量（AGA 推荐方案，CP3 已确认 A1/C1/E1）

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 | 状态 |
|---|---|---|---|---|---|---|---|---|
| DQ-BP-CR139-01 (AGA-1) | architecture | companion HLD 写侧/读侧分离是否分层同属 FEAT-02（HLD 分读写章），还是拆独立 FEAT-02R | A1 分层同属 FEAT-02 + HLD 分读写章 | A3 拆 FEAT-02R | A1 不增 Feature、可回退；A3 边界最清但增复杂度 | companion HLD 结构、Story owner、文件所有权 | CP5 读写冲突频繁 → 升级 A3 | confirmed-cp3 (A1/C1/E1, CP3 approved 2026-06-28T17:30:00+08:00) |
| DQ-BP-CR139-02 (AGA-3) | architecture | ML feature 层是否 lake `features/` 子层带版本（D5），保留切换独立 store 条件 | C1 lake features/ 子层 + DEF-139-01 切换条件 | C2 独立 feature store（deferred） | C1 不引入依赖、与 release 闭环一致；C2 在线 serving 友好但增依赖 | V3 物理归属、依赖、未来切换 | feature 规模/在线 serving 需求 → 切换 C2 另起 CR | confirmed-cp3 (A1/C1/E1, CP3 approved 2026-06-28T17:30:00+08:00) |
| DQ-BP-CR139-03 (AGA-5) | architecture | 配置类事实源版本化是否复用 V1 published pointer 语义 + 独立 config_facts 子目录 | E1 复用 V1 pointer 语义 + config_facts 子目录 | E2 各类独立 registry | E1 机制统一、减分叉；E2 量身定制但四套机制 | F1-F4 release 闭环、publish.py 复用 | policy_cycle 语义差异过大 → 该类独立，其余复用 | confirmed-cp3 (A1/C1/E1, CP3 approved 2026-06-28T17:30:00+08:00) |

### CR-139 蓝图自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 写侧/读侧分离已落 Feature 子域 + 数据归属 | PASS | §Feature / 数据归属增量（AGA-1 A1，CP3 已确认） |
| ML feature/读侧语义/run evidence/配置层 Feature 归属已补 | PASS | CAP-15..19、FEAT-02/03/06/11/12/14 增量 |
| 跨 Feature 流程写明 Owner 和失败路径 | PASS | FLOW-20..25 |
| 既有合同闭环非新建边界清晰 | PASS | REQ-249、§0 核验、FEAT-06/11/12/14 增量标注既有合同位置 |
| 运行授权边界未被放大 | PASS | REQ-248、不授权 runtime/NAS/QMT/trading/物理分区迁移/Git remote write |
| AGA 推荐方案 CP3 已确认 A1/C1/E1 | PASS | DQ-BP-CR139-01..03 状态 confirmed-cp3 |
