---
status: "draft-current-index"
version: "1.5"
source_use_cases: "process/USE-CASES.md"
source_requirements: "process/REQUIREMENTS.md"
source_story_backlog: "process/STORY-BACKLOG.md"
source_hld:
  - "process/HLD.md"
  - "process/HLD-DATA-LAKE.md"
  - "process/HLD-QMT-TRADING.md"
  - "process/archive/design-cr-docs/HLD-CR051-STRATEGY-RESEARCH-LIFECYCLE-FRAMEWORK.md"
source_adr: "process/ARCHITECTURE-DECISION.md"
change: "CR-138"
confirmed_by: ""
confirmed_at: ""
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

## 蓝图定位

| 项 | 当前口径 |
|---|---|
| 项目类型 | 本地 Python 量化研究、回测、数据湖、策略交付合同和交易接入准备工具 |
| 当前执行状态 | `process/STATE.md` 中 CR-046 保持 paused at CP6 / ready-for-verification；CR-051 已通过 CP3，正在进行 strategy research lifecycle / quant-lab migration 的 CP4 story-planning |
| 蓝图作用 | 为长期维护提供能力边界和数据归属索引，不替代 HLD、ADR、LLD 或检查点 |
| 授权边界 | 本文不授权真实 provider、lake、publish、gateway、QMT、MiniQMT、simulation、live、凭据读取、账户操作、NAS 扫描 / 挂载 / 搬迁、目录重命名、远端仓库改名、git push 或历史文件批量替换 |

## 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖 Story / CR | Owner Feature |
|---|---|---|---|---|
| CAP-01 | 本地离线研究与轻量日频回测 | 在断网或本地缓存条件下完成策略研究、参数扫描、候选报告和偏差披露 | STORY-001..013、CR-006、CR-008、CR-011、CR-025 | FEAT-01 |
| CAP-02 | 生产级市场数据湖 | 以 raw / manifest / canonical / gold / quality / catalog 形成可审计、可发布、可回滚的 current truth | CR-004、CR-005、CR-007、CR-010、CR-014、CR-017、CR-018 | FEAT-02 |
| CAP-03 | 多因子研究闭环 | 用本项目自有 FactorSpec / FactorRunSpec / FactorPanel / LabelWindow / report catalog 形成可复跑研究链路 | CR-011、CR-019、CR-030 | FEAT-03 |
| CAP-04 | 执行语义对齐与可选后端参考 | 把 lightweight engine、Backtrader optional semantic reference 和 order intent draft 的差异显式化 | CR-025、CR-030 | FEAT-04 |
| CAP-05 | QMT C/S Gateway 与只读运行准入 | 通过 Linux C 侧 client / CLI 和 Windows S 侧 gateway 打通受控只读查询准备 | CR-019、CR-020 | FEAT-05 |
| CAP-06 | OMS、风控、Broker Lake 与阶段激活 | 为后续 simulation / live_readonly / small_live / scale_up 提供订单状态、风控、对账和 kill switch 边界 | CR-015、CR-016、CR-017、CR-021..024 candidate | FEAT-06 |
| CAP-07 | 安全、授权与 no-real-operation 治理 | 保证 CP、Story verified、runbook 和 gateway health 不被误读为真实操作授权 | 全部 CR，重点 CR-014、CR-019、CR-020、CR-025、CR-030 | FEAT-07 |
| CAP-08 | 文档、Runbook 与发布证据 | 让用户能按当前安全边界操作、验证和恢复，同时保留审计证据 | README、docs/USER-MANUAL.md、docs/QMT-*.md、CP8 | FEAT-08 |
| CAP-09 | 双目标策略交付框架 | 让同一研究策略核心可按合同交付到 QMT 终端策略包和 MiniQMT runner 包，并由统一验证框架审查 | CR-046、CR047-candidate、CR049-candidate、CR051-candidate | FEAT-09 |
| CAP-10 | 策略研究生命周期与项目迁移治理 | 将 idea、资料、研究项目、运行证据、归档、策略交付候选和 quant-lab 迁移路径纳入统一生命周期与安全边界 | CR-051、CR052..CR056-candidate | FEAT-10 |

## Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 / 对象 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-01 | 本地研究与轻量回测核心 | `engine/`、`strategies/`、回测、扫描、候选、研究报告 metadata、只读 ResearchDataset 消费 | provider 抓取、真实 lake 写入、catalog publish、QMT native 调用、真实交易 | BacktestRun、PortfolioResult、MetricsReport、CandidateReport、BiasAudit | published catalog、quality/readiness、benchmark、factor panel、order intent draft 输入 | `market_data.connectors`、`market_data.runtime`、provider SDK、QMT / XtQuant、credential env |
| FEAT-02 | 生产级市场数据湖 | `market_data/` 生产链路、lake layout、dataset schema、run / manifest、normalization、validation、quality/readiness、catalog publish、replay、retention、rollback | 策略逻辑、因子评价算法、OMS、QMT 操作、broker lake | MarketDataRun、Manifest、DatasetCandidate、CatalogCurrentTruth、QualityReport、ReadinessReport、ClaimBoundary | CR / HLD / dataset requirements、source registry | `engine` 反向写入、trading、broker lake、QMT gateway |
| FEAT-03 | 研究数据集与多因子研究闭环 | `research_input_v1`、FactorSpec、FactorRunSpec、FactorPanel、LabelWindow、FactorEvaluationReport、MultiFactorCombiner、ExperimentManifest、ResearchReportCatalog、StrategyAdmissionPackage | 数据事实源写入、外部框架默认运行、真实 QMT / simulation / live、catalog publish | FactorSpec、FactorRunSpec、FactorPanelContract、LabelWindowSpec、ExperimentManifest、StrategyAdmissionPackage | FEAT-02 published data、FEAT-01 lightweight results、FEAT-04 semantic diff | provider、lake write、external project runtime、QMT gateway direct operation |
| FEAT-04 | 执行语义对齐与可选后端参考 | Backtrader optional semantic reference、clean feed gate、semantic diff schema、target portfolio / `order_intent_draft_v1` 衔接、license / no-copy guardrail | 默认回测主路径替代、源码迁移、真实 broker store、QMT live store、provider / lake / publish | SemanticDiffReport、BackendAvailability、OrderIntentDraft | clean feed、lightweight run output、data quality metadata | default dependency、source copy、real broker/QMT/provider |
| FEAT-05 | QMT C/S Gateway 与只读运行准入 | Linux C 侧 REST client / Typer CLI、Windows S 侧 gateway lifecycle、pairing/HMAC、allowlist/scope/nonce、QMT login/session ready、`query_positions` readonly endpoint | OMS 下单、simulation/live、账户写入、发单/撤单、broker lake 写入、策略准入判断 | GatewayConfig、PairingToken、HMACScope、QMTLoginSession、QueryPositionsResult | authorization record、redacted `.env` refs、endpoint matrix、stage gate status | C 侧导入 xtquant、gateway 默认公网暴露、无授权真实 QMT 调用 |
| FEAT-06 | OMS / 风控 / Broker Lake / 阶段激活 | target portfolio 到 order intent、OMS 状态机、pre-trade hard risk、broker lake schema、stage gate、reconciliation、kill switch、incident playbook | 因子生成、市场数据湖写入、gateway lifecycle、凭据存储 | OrderIntent、OMSOrder、BrokerEvent、BrokerLakeRecord、StageGate、AuthorizationRecord、KillSwitchEvent | FEAT-03 admission package、FEAT-05 gateway capabilities、FEAT-02 raw price / metadata | strategies 直连、复权价执行、无授权 adapter 调用 |
| FEAT-07 | 安全、授权与 no-real-operation 治理 | runtime authorization、risk acceptance、credential redaction、no-real-operation counters、forbidden import、human gate 不授权项 | 业务算法或数据 schema 本身 | AuthorizationDecision、NoRealOpCounter、RedactionPolicy、ForbiddenDependencyRule | 全部 Feature 的设计 / 测试 / 日志证据 | 任何通过文档或健康检查隐式升级运行授权的路径 |
| FEAT-08 | 文档、Runbook 与发布证据 | README、USER-MANUAL、QMT 安装 / 运行手册、incident playbook、release readiness、CP8 用户终验摘要 | 改变业务范围、授权真实操作、修改事实源 | Runbook、UserManualSection、ReleaseNote、RollbackPlan、FeedbackEntry | HLD / ADR / verification report / CP8 | 文档声明超出已验证状态或绕过 gate |
| FEAT-09 | QMT / MiniQMT 双目标策略交付框架 | 策略核心合同、策略包目录、QMT terminal target、MiniQMT runner target 安装设计、统一验证证据模型、后续策略交付门禁 | 具体策略交付、QMT 终端运行验证、MiniQMT 真实连接、真实安装、submit/cancel、simulation/live | StrategyCoreContract、StrategyPackageContract、QMTTerminalTargetContract、MiniQMTRunnerTargetContract、StrategyValidationEvidence | FEAT-03 StrategyAdmissionPackage、FEAT-04 OrderIntentDraft、FEAT-06 风控边界、FEAT-07 授权记录 | 策略核心直连 QMT / XtQuant、验证框架触发真实运行、安装设计读取凭据或写真实运行目录 |
| FEAT-10 | 策略研究生命周期与项目迁移治理 | InformationSource、StrategyIdea、ResearchProject、ResearchProtocol、ResearchRun、ValidationEvidence、ResearchArchiveManifest、ProjectIdentity、MigrationInventory、Follow-up CR roadmap | 具体策略实现、真实交易、provider/lake/publish、QMT/MiniQMT runtime、目录实际重命名、NAS 实际扫描 / 挂载 / 搬迁、远端仓库改名或 git push | InformationSource、StrategyIdea、ResearchProject、ResearchProtocol、ResearchRun、ValidationEvidence、ResearchArchiveManifest、StrategyTaxonomyEntry、ProjectIdentity、MigrationInventory | FEAT-02 catalog/data release、FEAT-03 admission package、FEAT-09 StrategyCoreContract / StrategyValidationEvidence | provider SDK、QMT / XtQuant / MiniQMT、broker lake write、真实 NAS 操作、凭据读取、git push / remote rename、批量重写历史 process 证据 |

## 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-01 | 用户显式授权数据生产 | FEAT-02 -> FEAT-01 / FEAT-03 | FEAT-02 | plan / run / normalize / validate 任一失败时不得 publish；consumer 返回 `required_missing` / `blocked_claims` | `tests/test_cr014_*`、`tests/test_cr018_*` |
| FLOW-02 | 研究运行需要生产数据 | FEAT-01 -> FEAT-02 | FEAT-02 | FEAT-01 只读 catalog，不自动补数；缺口返回 structured unavailable | `tests/test_cr010_consumer_boundary.py`、`tests/test_cr014_research_consumer_boundary.py` |
| FLOW-03 | 多因子研究闭环 | FEAT-03 -> FEAT-01 / FEAT-04 | FEAT-03 | factor / label / report 任一 gate 失败时 admission package blocked | `tests/test_cr030_*` |
| FLOW-04 | 研究输出转执行草稿 | FEAT-03 / FEAT-04 -> FEAT-06 | FEAT-04 / FEAT-06 | `order_intent_draft_v1` 不等于真实 order；缺 execution raw policy 时 blocked | `tests/test_cr025_order_intent_draft_contract.py`、`tests/test_cr030_strategy_admission_package.py` |
| FLOW-05 | QMT 只读实机验证 | FEAT-05 -> FEAT-07 -> FEAT-08 | FEAT-05 | 缺 MiniQMT 权限、凭据、pairing、scope 或 session ready 时 blocked，不关闭 CR-020 | `tests/test_cr020_*` + 用户脱敏 evidence |
| FLOW-06 | 后续 simulation / live 路线 | FEAT-03 -> FEAT-06 -> FEAT-05 -> FEAT-07 | FEAT-06 | CR-021..024 未启动 / 未授权时全部 blocked；gateway capabilities 不升级授权 | `tests/test_cr016_*`、后续 CR |
| FLOW-07 | 文档发布和终验 | FEAT-08 -> FEAT-07 | FEAT-08 | README / runbook / CP8 不得声明未授权真实能力 | docs guardrail tests、CP8 |
| FLOW-08 | 研究策略进入双目标交付 | FEAT-03 / FEAT-04 -> FEAT-09 -> FEAT-07 | FEAT-09 | 缺少策略核心合同、order intent draft、风险假设或验证证据时，策略包不得进入 CR047 交付 | CR046 CP3/CP5、后续 CR047 |
| FLOW-09 | MiniQMT runner 安装设计审查 | FEAT-09 -> FEAT-07 -> FEAT-08 | FEAT-09 | 本 CR 只允许 install dry-run 设计；真实安装、连接和运行必须进入 CR049 / runtime authorization gate | CR046 验证框架、后续 CR049 |
| FLOW-10 | 策略想法进入研究闭环 | FEAT-10 -> FEAT-03 -> FEAT-09 -> FEAT-07 | FEAT-10 / FEAT-03 / FEAT-09 | idea、protocol、run、validation 任一证据缺失时只能保持 research-only 或 blocked，不得升级为 delivery_candidate | CR051 CP5/CP7、后续 CR052 |
| FLOW-11 | 项目身份和仓库结构迁移 | FEAT-10 -> FEAT-08 -> FEAT-07 | FEAT-10 | 仅在 inventory、Git archive、mechanical move plan、legacy alias 验证和用户授权齐备后才允许真实迁移；CP4/CP5 只做设计 | CR051 CP4/CP5、后续迁移授权门禁 |

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

## 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-BP-001 | follow_up_tracking | 是否把 CR-031 的蓝图索引作为后续 CR-021..024 的默认设计入口 | 推荐：作为默认入口，但不替代 legacy HLD/ADR 审计 | 仅保留为临时说明 | 推荐方案能降低后续翻阅成本；备选不改变现状但边界继续分散 | 影响后续 QMT simulation/live 设计交接 | 若发现索引与 legacy 事实冲突，以 legacy HLD/ADR/Story 为准并修订蓝图 |
| DQ-BP-002 | implementation | 是否立即回写 134 个 Story 的 `feature_design_refs` | 推荐：本轮不批量回写，只在后续变更 Story 中引用矩阵 | 批量回写所有 Story | 推荐方案低风险；批量回写会污染已验证证据且容易引入误差 | 影响 Story 历史审计和 git diff 规模 | 后续新 CR 或重开 Story 时增量回写 |
| DQ-BP-CR046-01 | architecture | CR046 是否新增独立 FEAT-09 承载双目标策略交付框架，而不把能力并入 FEAT-05 / FEAT-06 | 推荐：新增 FEAT-09，FEAT-05/06 仍保留 gateway 和交易治理边界 | 并入 FEAT-05；并入 FEAT-06 | 推荐方案避免把“策略交付合同”误读为 gateway runtime 或 OMS 授权；并入方案更少 Feature 但边界混淆 | 影响 CR047 策略包和 CR049 runner install 的消费入口 | 若后续只保留 QMT terminal 且放弃 MiniQMT，可将 FEAT-09 降级为 FEAT-05/06 的子能力 |
| DQ-BP-CR051-01 | architecture | CR051 是否新增 FEAT-10 承载策略研究生命周期和 quant-lab 迁移治理 | 推荐：新增 FEAT-10，FEAT-03 继续负责因子研究和 admission，FEAT-09 继续负责策略交付包 | 并入 FEAT-03；并入 FEAT-08 | 推荐方案能把研究 lifecycle、archive、项目身份和迁移边界集中管理；并入 FEAT-03 会扩大研究算法边界，并入 FEAT-08 会把迁移治理误降级为文档刷新 | 影响 CR052..CR056 的进入门禁和本项目迁移顺序 | 若 CR051 CP5 发现 FEAT-10 只剩文档更新，可降级为 FEAT-08 technical-note；若真实迁移启动，仍需独立 runtime_authorization / migration gate |

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 每个 Feature / Epic 有职责、非职责和数据归属 | PASS | §Feature / Epic 边界 |
| 跨 Feature 流程写明 Owner 和失败路径 | PASS | §跨 Feature 流程 |
| 共享能力写明调用方向和降级策略 | PASS | §共享能力 |
| 运行授权边界未被蓝图放大 | PASS | §蓝图定位、FEAT-07、FEAT-09、FEAT-10、DQ-BP-CR046-01、DQ-BP-CR051-01 |

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
