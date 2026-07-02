---
status: "ready-for-cp5-review"
version: "1.15"
source_blueprint: "docs/design/BLUEPRINT.md"
source_hld:
  - "docs/design/HLD.md"
  - "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
source_adr:
  - "docs/design/ARCHITECTURE-DECISION.md"
  - "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
change: "CR-152"
companion_hld_cr139: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Design Matrix

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-07 | meta-po | 新增 Feature 设计适用性矩阵，覆盖八个主要 Feature / Epic |
| 1.1 | 2026-06-13 | meta-po | 按 CR-046 增补 FEAT-09 双目标策略交付框架，实现设计三件套和 7 个 Story 的 lld_policy |
| 1.2 | 2026-06-14 | host-orchestrator | 按 CR-051 增补 FEAT-10 策略研究生命周期与 quant-lab 迁移治理，实现设计三件套和 6 个 Story 的 lld_policy |
| 1.3 | 2026-06-14 | host-orchestrator | 按 CR-053 增补 FEAT-10-CR053 quant-lab migration inventory / dry-run scoped design、5 个 Story 和 CP5 批次 |
| 1.4 | 2026-06-23 | host-orchestrator | CR131 将 CR 命名 feature design matrix 历史文件移入 archive；默认 design root 保留当前矩阵。 |
| 1.5 | 2026-06-24 | host-orchestrator | 按 CR138 CP3 approved HLD 增补 FEAT-11 Runner Control Plane 与 FEAT-12 QMT Gateway Service Layer，实现设计三件套和 8 个 Story 的 lld_policy；复用 FEAT-07 / FEAT-06 作为 shared safety / OMS 输入。 |
| 1.6 | 2026-06-27 | codex | FEAT-03 增补统一因子目录、查询 CLI 和 mature runner 因子纳管；重访条件增加新增因子、mature runner 因子、查询 CLI 和目录 schema 变化。 |
| 1.7 | 2026-06-27 | codex | FEAT-03 增补高级多因子模型评估报告、mature admission 门禁和政策周期配置合同；重访条件增加评估 schema、gate policy、样本外 / GRS / 做空 / 壳价值 / 政策周期评估变化。 |
| 1.8 | 2026-06-27 | codex | FEAT-03 / FEAT-14 收紧 Stage 3 出口：必须输出评估通过的成熟多因子策略；新增 candidate sweep、pass gate 和 blocked 候选不得出阶段的重访条件。 |
| 1.9 | 2026-06-27 | codex | FEAT-03 增补异象发现 / 研究设计入口；重访条件增加 AnomalyCandidate、Harvey 标准、分组单调性、控制因子 alpha、时间切分、成本和经济逻辑准入变化。 |
| 1.10 | 2026-06-27 | codex | FEAT-03 增补自动异象发现系统；重访条件增加受控模板、批量 discovery runner、多重检验、动态因子目录接入和 Stage 3 候选消费变化。 |
| 1.11 | 2026-06-28 | codex | FEAT-03 增补研究引擎稳定模块整改；重访条件增加 engine 主入口命名、旧 engine 入口归档、公共 serialization / matrix / admission contracts 和脚本归档规则变化。 |
| 1.12 | 2026-06-28 | meta-se | 按 CR-139「Strategy Data Foundation」CP3 增补策略生产数据底座 Feature 归属与 `lld_policy`：FEAT-02 写侧/读侧分层（V1/C1/C2/R1/R3/R4/L2/L4/L5/M1/M2/N1-N3/C3/C4/T7/T8/X1/X2/X3/V4/F1）、FEAT-03 ML feature 层（V3/R2/E1-E5/F4）、FEAT-06 交易审计链（T4/T6）、FEAT-11 run evidence（L3/T6）、FEAT-12 配置层（F2/T5）、FEAT-14 universe·risk policy（F3/X4）；`lld_policy`：d1 纯新建=full-lld、d2 既有合同闭环=technical-note、a 已设计未实现=technical-note 消费既有 HLD 契约；跨边界项 T6 run-id 贯通归 FEAT-02 写侧生成 + FEAT-06/11 消费。AGA-1/3/5 推荐方案 CP3 已确认 A1/C1/E1。 |
| 1.13 | 2026-06-28 | host-orchestrator | CP3 approved，AGA-1/3/5 确认 A1/C1/E1，pending-cp3 → confirmed-cp3。 |
| 1.14 | 2026-07-01 | host-orchestrator | CR151 CP3 approved 后增补 Strategy Admission Statistical Gate CP4：FEAT-03 下新增统计准入门 Story 消费、`lld_policy`、DAG/Wave 和 Wave B deferred 边界。 |
| 1.15 | 2026-07-02 | host-orchestrator | CR152 CP3 approved 后增补 ML Strategy E2E Framework CP4：FEAT-03 下新增 ML first-wave Story 消费、`lld_policy`、DAG/Wave、triple_barrier BLOCKED CP5 约束和 no-registry-write 边界。 |

## 适用性判定规则

| 维度 | 需要 Feature 设计的触发条件 | 可豁免条件 |
|---|---|---|
| 数据与状态 | 新增 / 修改核心对象、状态机、迁移、兼容策略 | 只读展示或无状态配置 |
| 接口与依赖 | 跨模块、外部接口、共享契约、依赖方向需要冻结 | 单文件局部改动且无新接口 |
| 权限与安全 | 权限边界、敏感信息、审计、运行授权 | 无权限变化且无敏感数据 |
| 运行与可靠性 | 并发、幂等、重试、性能、降级、回滚 | 无运行时风险扩展 |
| 多 Story 复用 | 多个 Story 共享同一能力边界或任务清单 | 单 Story 可直接用技术说明覆盖 |

## Feature 设计矩阵

| Feature ID | Feature / Epic | 来源 | 适用性 | 判定理由 | 需要产物 | 关联 Story | 建议 lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|---|
| FEAT-01 | 本地研究与轻量回测核心 | BLUEPRINT、HLD、STORY-001..013 | waived | 基线已 verified，当前不重写已验证 Story；后续新增高风险改造再进入 required | none / legacy evidence | STORY-001..013、CR008、CR011 | waived for baseline；新增跨模块改造用 full-lld | 修改 data loader、backtest engine、portfolio 或 report schema 时 |
| FEAT-02 | 生产级市场数据湖 | BLUEPRINT、HLD-DATA-LAKE、ADR-013..022/030..035/048..054/062..066 | required | 拥有事实源、publish gate、rollback、external lake root、provider 授权和 schema 状态机 | `docs/features/market-data-lake/*` | CR004..CR005、CR007、CR010、CR014、CR017、CR018 | full-lld | 新增 dataset、publish 规则、DuckDB 事实边界、真实写湖或 rollback 行为时 |
| FEAT-03 | 研究数据集与多因子研究闭环 | BLUEPRINT、HLD §35、ADR-079..086 | required | 多个 Story 共享 FactorCatalogEntry、AnomalyCandidate、AnomalyDiscoveryRun、FactorSpec、FactorRunSpec、LabelWindow、FactorModelValidationReport、ReportCatalog、AdmissionPackage 和研究引擎稳定模块；Stage 3 出口必须由评估通过的成熟策略关闭 | `docs/features/factor-research-loop/*` | CR011、CR019、CR030、factor-catalog-cli、factor-model-validation、anomaly-discovery-research、automatic-anomaly-discovery、research-engine-stable-modules、stage3-pass-candidate | full-lld | 新增因子、mature runner 因子、查询 CLI、目录 schema、engine 主入口命名、旧 engine 入口归档、公共 serialization / matrix / admission contracts、受控异象模板、自动 discovery runner、多重检验策略、动态目录接入、Stage 3 anomaly candidate spec、异象候选 / 研究报告 schema、Harvey 标准、分组单调性、控制因子 alpha、时间切分、成本 / 换手、经济逻辑准入、评估报告 schema、gate policy、GRS、因子溢价、经济显著性、样本外、做空、壳价值、政策周期、label window、组合器、admission package 字段、candidate sweep 或 Stage 3 pass gate 时 |
| FEAT-04 | 执行语义对齐与可选后端参考 | BLUEPRINT、HLD §34、ADR-074..078 | required | 涉及 optional backend、依赖隔离、license/no-copy、semantic diff 和 order intent draft | `docs/features/execution-semantics-reference/*` | CR025、CR030 | full-lld | 新增 Backtrader / Qlib / external runner 依赖、运行或源码适配时 |
| FEAT-05 | QMT C/S Gateway 与只读运行准入 | BLUEPRINT、HLD §36、ADR-087..093 | required | 跨 Linux / Windows、HMAC、凭据引用、QMT login/session 和只读真实验证风险高 | `docs/features/qmt-gateway-readonly/*` | CR019、CR020 | full-lld | 恢复 CR-020 真实验证、扩大 endpoint 或引入新 gateway runtime 时 |
| FEAT-06 | OMS / 风控 / Broker Lake / 阶段激活 | BLUEPRINT、HLD-QMT、ADR-055..061 | required | 状态机、风控、broker facts、真实交易阶段和 kill switch 风险最高 | `docs/features/qmt-trading-governance/*` | CR015、CR016、CR017、CR021..024 candidate | full-lld | 启动 CR-021 simulation、CR-022 live_readonly、CR-023 small_live、CR-024 scale_up 时 |
| FEAT-07 | 安全、授权与 no-real-operation 治理 | BLUEPRINT、DEPENDENCY-MAP、全部高风险 CR | required | 横切 provider/lake/publish/QMT/credential/authorization，必须有统一测试与文档门禁 | `docs/features/runtime-authorization-safety/*` | CR014、CR019、CR020、CR025、CR030、后续 CR021..024 | full-lld | 任一真实操作授权、凭据路径、日志策略或 CP 人工门禁语义变化时 |
| FEAT-08 | 文档、Runbook 与发布证据 | README、USER-MANUAL、QMT docs、CP8 | waived / required-by-change | 普通文档刷新可 waived；涉及授权语义、运行手册、真实验证步骤时 required | none by default；按 CR 生成 docs plan | CR015..CR020、CP8 | technical-note / full-lld when safety relevant | 文档新增真实运行步骤、授权声明或 runbook 流程时 |
| FEAT-09 | QMT / MiniQMT 双目标策略交付框架 | BLUEPRINT v1.1、archived HLD-CR046、ADR-CR046-001..006 | required | legacy cross-target framework；offline runner implementation authority 已转到 `strategy-runner-core`；涉及外部交易终端边界与 no-real-operation 安全约束 | `docs/features/qmt-miniqmt-dual-target-framework/*` | CR046-S01..S07 | full-lld for S01..S05；technical-note for S06..S07 | 启动具体策略交付、真实 QMT shadow、MiniQMT install / connection、submit/cancel 或研究框架反向完善时 |
| FEAT-10 | 策略研究生命周期与项目迁移治理 | BLUEPRINT v1.3、DOMAIN-MAP v1.2、DEPENDENCY-MAP v1.2、archived HLD-CR051 | required | 新增策略生命周期、taxonomy、archive manifest、硬件冷热分层、项目身份、迁移 inventory 和后续 CR gate；涉及迁移、安全和跨 Feature 合同 | `docs/features/strategy-research-lifecycle/*` | CR051-S01..S06 | full-lld for S01..S04；technical-note for S05..S06 | 启动 CR052 多因子完整证明、真实目录迁移、NAS 操作、项目包名重命名、交易主机 package 消费或 runtime_candidate gate 时 |
| FEAT-10-CR053 | quant-lab migration inventory / dry-run | archived HLD-CR053、ADR-CR053-001..007、CR051 archive governance | required | CR053 是 FEAT-10 的迁移 dry-run 增量，新增 root map、repo inventory、path references、transfer/backup plan 和 CR058 输入；涉及 NAS / lake / Windows / Linux 映射和不授权边界 | `docs/features/quant-lab-migration-dry-run/*` | CR053-S01..S05 | full-lld for S01..S04；technical-note for S05 | CR058 真实 repo-local migration、CR060 NAS/archive 实迁、数据湖 root 迁移、交易机 package import 方式变化时 |
| FEAT-11 | Runner Control Plane | CR138 HLD、ADR-CR138-001/003/005/007、BLUEPRINT v1.5、DOMAIN-MAP v1.4 | required | 新增运营控制面，负责 RunPlan、PreflightResult、RunnerCommand、RunState、RunEvidence、ReviewSummary、IncidentRecord 和 StrategyChangePlan；涉及状态机、授权边界、事件幂等、审计和跨 Gateway / OMS 契约 | `docs/features/runner-control-plane/*` | CR138-S01..S04、CR138-S08 | full-lld for S01..S04/S08 | 新增 Runner 运营命令、事件接入、盘前检查、复盘、异常恢复、策略变更或 runtime authorization 消费时 |
| FEAT-12 | QMT Gateway Service Layer | CR138 HLD、ADR-CR138-002/003/004/005/006/007、BLUEPRINT v1.5、DOMAIN-MAP v1.4 | required | 新增 Gateway 服务层，负责 REST-only P0、GatewayHealth、CapabilitySnapshot、TradingSession、TradingCalendar、CommissionSchedule、PnLSnapshot、MarketSubscription、GatewayCommand、ExecutionReport、AuditRecord 和 ChangePlan；涉及外部接口、查询授权、市场订阅、订单回报、恢复和审计 | `docs/features/qmt-gateway-service-layer/*` | CR138-S01、CR138-S05..S08 | full-lld for S01/S05..S08 | 新增 Gateway endpoint、账户 / 行情 / 订单只读、订阅、order write、恢复、协议扩展或配置变更时 |

## Story 下游消费表

| Story / Story Group | feature_design_refs | lld_policy.required_level | trigger_reasons | 设计证据 | CP5 审查方式 |
|---|---|---|---|---|---|
| STORY-001..013 | legacy HLD / ADR / Story LLD | waived for CR-031 | baseline verified | `process/stories/STORY-*.md` | 不回写历史 Story |
| CR004..CR018 data-lake-related | `docs/features/market-data-lake/DESIGN.md` | full-lld for new changes | data / state / publish / migration / authorization | existing `process/stories/CR*-LLD.md` + feature index | 后续 CR 增量审查 |
| CR030-S01..S08 | `docs/features/factor-research-loop/DESIGN.md` | full-lld | schema / report catalog / admission / no-real-op | existing CR030 LLD + feature index | 后续变更审查 |
| CR025-S01..S06 | `docs/features/execution-semantics-reference/DESIGN.md` | full-lld | optional backend / license / semantic diff | existing CR025 LLD + feature index | 后续变更审查 |
| CR019 / CR020 | `docs/features/qmt-gateway-readonly/DESIGN.md` | full-lld | external runtime / security / readonly QMT | CR019 / CR020 LLD + feature index | CR020 恢复前审查 |
| CR015 / CR016 / future CR021..024 | `docs/features/qmt-trading-governance/DESIGN.md` | full-lld | OMS / risk / stage gate / broker lake | CR015 / CR016 LLD + feature index | 新 CR 必须审查 |
| 全部高风险 CR | `docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | runtime authorization / no-real-op / redaction | safety feature index | CP2/CP3/CP5/CP8 均消费 |
| CR046-S01-dual-target-strategy-architecture | `docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md` | full-lld | architecture / cross-feature / safety | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR046-S02-strategy-package-contract-and-schema | `docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md`、`TEST-PLAN.md` | full-lld | schema / contract / validation | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR046-S03-qmt-terminal-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | external terminal boundary / no-runtime | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR046-S04-miniqmt-runner-install-and-runtime-boundary | `docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | install design / external runtime boundary / credential safety | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR046-S05-verification-framework-and-evidence-model | `docs/features/qmt-miniqmt-dual-target-framework/TEST-PLAN.md` | full-lld | validation evidence / safety claims | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR046-S06-follow-up-strategy-delivery-gate | `docs/features/qmt-miniqmt-dual-target-framework/TASKS.md` | technical-note | follow-up tracking / low implementation risk | Story 技术说明 | CP5 自动预检 + 批量人工确认 |
| CR046-S07-research-framework-follow-up-contract | `docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md` | technical-note | research handoff / follow-up contract | Story 技术说明 | CP5 自动预检 + 批量人工确认 |
| CR051-S01-lifecycle-and-taxonomy-framework | `docs/features/strategy-research-lifecycle/DESIGN.md`、`TEST-PLAN.md` | full-lld | lifecycle / taxonomy / cross-feature / claim boundary | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR051-S02-repository-archive-and-data-lake-governance | `docs/features/strategy-research-lifecycle/DESIGN.md`、`TEST-PLAN.md` | full-lld | archive governance / storage tiering / lake boundary / safety | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR051-S03-research-pc-and-trading-pc-workflow | `docs/features/strategy-research-lifecycle/DESIGN.md`、`TASKS.md` | full-lld | host workflow / package consumer boundary / migration safety | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR051-S04-registry-and-evidence-contracts | `docs/features/strategy-research-lifecycle/DESIGN.md`、`TEST-PLAN.md` | full-lld | manifest schema / validation evidence / guardrail | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR051-S05-follow-up-cr-roadmap-and-admission-gates | `docs/features/strategy-research-lifecycle/TASKS.md` | technical-note | follow-up tracking / admission gate | Story 技术说明 | CP5 自动预检 + 批量人工确认 |
| CR051-S06-project-identity-rename-and-legacy-alias | `docs/features/strategy-research-lifecycle/DESIGN.md`、`TASKS.md` | technical-note | project identity / alias compatibility / no bulk rewrite | Story 技术说明 | CP5 自动预检 + 批量人工确认 |
| CR053-S01-root-map-and-host-mapping-contract | `docs/features/quant-lab-migration-dry-run/DESIGN.md`、`TEST-PLAN.md` | full-lld | NAS root map / Linux research host / Windows package exchange / lake alias | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR053-S02-repo-inventory-and-path-classification | `docs/features/quant-lab-migration-dry-run/DESIGN.md`、`TEST-PLAN.md` | full-lld | repo-local inventory / path classification / forbidden content boundary | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR053-S03-path-reference-and-legacy-alias-dry-run | `docs/features/quant-lab-migration-dry-run/DESIGN.md`、`TEST-PLAN.md` | full-lld | path references / legacy alias / manual-review dry-run | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR053-S04-manifest-transfer-and-backup-plan | `docs/features/quant-lab-migration-dry-run/DESIGN.md`、`TEST-PLAN.md` | full-lld | manifest-first transfer / backup plan / restore rehearsal | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR053-S05-cr058-migration-input-and-close-gate | `docs/features/quant-lab-migration-dry-run/DESIGN.md`、`TASKS.md` | technical-note | CR058 input / close gate / no-real-migration boundary | Story 技术说明 | CP5 自动预检 + 批量人工确认 |
| CR138-S01-shared-contracts-authorization-audit | `docs/features/runner-control-plane/DESIGN.md`、`docs/features/qmt-gateway-service-layer/DESIGN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | shared command/event/audit/auth contract / security boundary | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR138-S02-runner-plan-preflight-control | `docs/features/runner-control-plane/DESIGN.md`、`TEST-PLAN.md` | full-lld | RunPlan / PreflightResult / RunnerCommand / no runtime auth | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR138-S03-runner-event-signal-rebalance-tracking | `docs/features/runner-control-plane/DESIGN.md`、`docs/features/qmt-trading-governance/DESIGN.md` | full-lld | event ingestion / idempotency / rebalance / run tracker / OMS risk handoff | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR138-S04-runner-evidence-review-incident-lifecycle | `docs/features/runner-control-plane/DESIGN.md`、`docs/features/runtime-authorization-safety/TEST-PLAN.md` | full-lld | evidence redaction / review / incident / recovery / strategy change plan | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR138-S05-gateway-lifecycle-health-rest-contract | `docs/features/qmt-gateway-service-layer/DESIGN.md`、`TEST-PLAN.md` | full-lld | lifecycle / health / capabilities / REST-only P0 / session boundary | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR138-S06-gateway-query-calendar-commission-pnl | `docs/features/qmt-gateway-service-layer/DESIGN.md`、`docs/features/qmt-trading-governance/DESIGN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | TradingCalendar / CommissionSchedule / PnLSnapshot / account readonly auth / redaction | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR138-S07-gateway-subscription-order-report-recovery | `docs/features/qmt-gateway-service-layer/DESIGN.md`、`docs/features/qmt-trading-governance/DESIGN.md` | full-lld | market subscription / GatewayCommand / ExecutionReport / recovery / hard reject | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR138-S08-docs-fixtures-cp7-authorization-runbook | `docs/features/runner-control-plane/TASKS.md`、`docs/features/qmt-gateway-service-layer/TASKS.md`、`docs/features/runtime-authorization-safety/TEST-PLAN.md` | full-lld | docs / fixture matrix / no-real-op guardrail / CP7 auth boundary | Story LLD | CP5 自动预检 + 批量人工确认 |

## 提前确认的关键决策

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 | 状态 |
|---|---|---|---|---|---|---|---|---|
| DQ-FD-001 | implementation | 是否为所有 required Feature 补完整 DESIGN / TEST-PLAN / TASKS | 已按推荐补齐 FEAT-02、FEAT-03、FEAT-04、FEAT-05、FEAT-06、FEAT-07 六个 required Feature 索引 | 只补蓝图三件套，不补 Feature 索引 | 推荐方案让后续 CR 有稳定设计入口；备选 diff 更小但仍需翻 legacy HLD | 影响后续 CR 的设计入口完整性 | 若后续 Feature 设计与 legacy HLD 冲突，以 legacy HLD/ADR 为准并修订索引 | resolved-cr031 |
| DQ-FD-002 | implementation | 是否批量修改历史 Story 卡片增加 `feature_design_refs` | 推荐：不批量修改，只在后续变更 Story 中增量引用 | 批量回写 134 个 Story | 推荐方案避免污染已验证历史证据；批量回写追溯更完整但风险高 | 影响审计 diff 和 Story 历史稳定性 | 若未来执行统一文档迁移 CR，可专门处理 | resolved-cr031 |
| DQ-FD-CR046-01 | implementation | 是否为 FEAT-09 生成独立 DESIGN / TEST-PLAN / TASKS | 已生成；作为 CR046-S01..S07 的共同设计输入 | 只在 HLD 中保留；拆成多个 Feature 目录 | 推荐方案让策略包、QMT target、MiniQMT runner 和验证框架共用同一入口；只放 HLD 会让 CP5 LLD 输入分散 | 影响 CP4、CP5 和后续 CR047/049/051 消费 | 若后续 FEAT-09 变大，可拆分子 Feature，但需新 CR 或 CP5 决策 | resolved-cp4 |
| DQ-FD-CR046-02 | implementation | 策略从研究侧传到交易运行 PC 的默认形态是什么 | 采用 `strategy-package-<strategy_id>-<version>.zip` + `.sha256` + `manifest.yaml`，经人工/受控文件通道传到交易运行 PC，再由 QMT terminal target 人工导入 | Git release / 内网共享目录 / U 盘离线交付 / 自动同步 runner | 推荐方案可审计、可校验、可回滚，并隔离研究环境与交易 PC；自动同步 runner 会引入运行授权风险 | 影响 CR046-S02/S03 的 artifact、checksum、transfer_channel 和 manual_import_steps 字段 | 若 CP5 发现交易 PC 环境约束不同，可把 transfer_channel 设为枚举，但不得自动运行 | approved-by-user-2026-06-13 |
| DQ-FD-CR051-01 | implementation | 是否为 FEAT-10 生成独立 DESIGN / TEST-PLAN / TASKS，并将 CR051-S01..S06 纳入单一 CP5 批次 | 生成独立三件套；S01..S04 full-lld，S05..S06 technical-note；统一批次 `CR051-STRATEGY-RESEARCH-LIFECYCLE-BATCH-A` | 只保留 HLD；拆成迁移 Feature 与研究 Feature 两个目录 | 推荐方案降低 CP5 下游读取成本，并把迁移 / archive / lifecycle 的安全边界合并审查；只留 HLD 会让 Story LLD 输入分散，拆两个 Feature 会增加当前 CP4 复杂度 | 影响 CR052..CR056 进入条件、项目迁移计划和后续文档刷新 | 若 CP5 发现迁移实施风险高于研究生命周期，可拆出后续 Migration CR；真实迁移仍需独立授权 | resolved-cp4 |
| DQ-FD-CR053-01 | implementation | 是否为 CR053 生成 FEAT-10 scoped migration dry-run 三件套，并将 CR053-S01..S05 纳入单一 CP5 批次 | 生成 `docs/features/quant-lab-migration-dry-run/*`；S01..S04 full-lld，S05 technical-note；统一批次 `CR053-MIGRATION-INVENTORY-BATCH-A` | 只复用 FEAT-10 CR051 三件套；拆成 NAS / repo / backup 三个 Feature | 推荐方案让 CR053 dry-run 输入独立可审查，同时不污染 CR051 closed baseline；只复用 CR051 会让迁移 dry-run 细节分散；拆三 Feature 对当前范围过重 | 影响 CP5 设计证据、CP6 静态报告和 CR058 输入 | 若 CP5 发现 NAS / 数据湖迁移超出 dry-run，可拆出 CR060+；真实迁移仍需独立授权 | resolved-cp4 |
| DQ-FD-CR138-01 | implementation | 是否为 FEAT-11 / FEAT-12 生成独立 DESIGN / TEST-PLAN / TASKS，并将 CR138-S01..S08 纳入单一 CP5 批次 | 生成 `docs/features/runner-control-plane/*` 与 `docs/features/qmt-gateway-service-layer/*`；8 个 Story 全部 full-lld；统一批次 `CR138-RUNNER-QMT-OPERATIONAL-CONTROL-BATCH-A` | 只保留 HLD；拆成 Runner / Gateway 两个 CP5 批次 | 推荐方案让 Runner/Gateway 共享合同和授权边界统一确认；只留 HLD 会让 Story LLD 输入分散；拆双批次会增加跨契约漂移 | 影响 CP5 设计证据、Story owner、Wave 和 runtime authorization 边界 | 若 CP5 发现批次过大，可在不改 HLD 的前提下拆成 Runner / Gateway 两个 CP5 子批次 |
| DQ-FD-CR138-02 | implementation | CR138-S08 文档 / fixtures / guardrail Story 是否需要 full-lld | 推荐 full-lld，因为它承载 no-real-op、runtime_authorization、docs 声明和 CP7 验证矩阵 | technical-note | 推荐方案降低误授权风险；technical-note token 更少但可能遗漏禁止项和验证命令 | 影响 CP7 / CP8 release readiness 和用户 runbook 语义 | 若 CP5 明确只做纯文档索引刷新，可降级为 technical-note |
| DQ-FD-CR138-03 | runtime_authorization | CP4 / CP5 Story 设计是否授权真实 QMT / 账户 / 行情 / 订单验证 | 推荐不授权；仅记录后续可按需申请 scoped runtime_authorization | CP5 直接授权 readonly；永久不授权 | 推荐方案保持 CP3 决策一致，同时保留后续验证路径；直接授权风险高，永久不授权会阻断必要验证 | 影响 CP6/CP7 验证模式和用户预期 | 任一真实运行前必须另起 runtime_authorization gate |

## CR151 CP4 增量：Strategy Admission Statistical Gate

> 来源：`process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md`、`process/docs/design/ARCHITECTURE-DECISION-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md`、CP3 用户批准。CR151 归属 FEAT-03「研究数据集与多因子研究闭环」的本地/static 统计准入补齐，不新增生产数据湖、NAS、runtime 或交易授权。

### Feature 归属与 lld_policy

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 | 说明 |
|---|---|---|---|---|---|---|
| CR151-S01-statistical-report-contracts | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | 新共享 contract module、typed report objects、status enum、JSON-safe serialization | Story LLD | 定义 MultipleTestingReport、RobustFactorStatisticsReport、WalkForwardValidationPlan、BacktestOverfitRiskReport、StrategyAdmissionStatisticalGate 的最小合同。 |
| CR151-S02-gate-evaluator-fail-closed-rules | FEAT-03 / FEAT-07 safety boundary | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md` | full-lld | fail-closed admission semantics、四态模型、forbidden operation counters | Story LLD | 覆盖 PASS/FAIL/NEEDS_REVIEW/BLOCKED，mandatory missing 必须 BLOCKED。 |
| CR151-S03-admission-completion-linkage | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TASKS.md` | full-lld | 触碰既有 CR150 completion map / StrategyAdmissionPackage linkage | Story LLD | 只做统计 gate ref / blocked reason linkage，不改变 runtime authorization 语义。 |
| CR151-S04-static-evidence-release-wording | FEAT-03 / FEAT-08 | `docs/features/factor-research-loop/TEST-PLAN.md`、`TASKS.md` | technical-note | CP6/CP7/CP8 evidence wording、static-only release boundary | Story technical note | 收口 evidence index、return packet、release wording 和 no-real-operation 声明。 |

## CR152 CP4 增量：ML Strategy E2E Framework

> 来源：`process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md`、`process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md`、CP3 用户批准。CR152 归属 FEAT-03「研究数据集与多因子研究闭环」的 ML strategy first-wave foundation，不新增真实训练、真实数据验证、model registry write、store write、production catalog pointer 或 runtime 授权。

### Feature 归属与 lld_policy

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 | 说明 |
|---|---|---|---|---|---|---|
| CR152-S01-pit-feature-label-contracts | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | PIT feature matrix、label policy、leakage guard、triple_barrier / meta_label reserved slots | Story LLD | CP5 必须明确 first-wave active `triple_barrier` enforcement；推荐直接 `BLOCKED`，因为算法未实现。 |
| CR152-S02-purged-embargo-cv-fixture-contract | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md` | full-lld | purged + embargo CV split policy、fixture schema、fold overlap / embargo failure paths | Story LLD | Fixture 必须可证明最小时间跨度、PIT 模拟和 fail-closed negative cases。 |
| CR152-S03-training-model-prediction-metadata | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TASKS.md` | full-lld | `TrainingSnapshotSpec` / `ModelArtifactRef` / prediction artifact metadata、contract delta mapping、no registry write | Story LLD | Metadata/ref/hash/linkage only；不得实现 registry writer、publish、promote、upload、set_current 或 catalog pointer mutation。 |
| CR152-S04-ml-admission-gate-adapter | FEAT-03 / FEAT-07 safety boundary | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | ML-specific admission gate、CR151 四态 status adapter、StrategyAdmissionPackage linkage、forbidden operation counters | Story LLD | 若 adapter 字段映射需要架构深化，CP5 应考虑 meta-se dispatch 或显式 inline-fallback。 |
| CR152-S05-static-evidence-release-wording | FEAT-03 / FEAT-08 | `docs/features/factor-research-loop/TEST-PLAN.md`、`TASKS.md` | technical-note | CP6/CP7/CP8 evidence wording、fixture-only claim boundary、no real model performance wording | Story technical note | 收口 evidence index、return packet、release wording 和 no-real-training/no-registry 声明。 |

### CR152 First-Wave / Later-Wave 边界

| 范围 | 状态 | 处理 |
|---|---|---|
| PIT feature matrix contract | First wave | CR152 必做。 |
| Label policy / leakage guard | First wave | CR152 必做；`fixed_window` active，`triple_barrier` / `meta_label` 只预留 slot。 |
| Purged + embargo CV split policy and split audit | First wave | CR152 必做。 |
| Training snapshot / model artifact metadata | First wave | CR152 必做；metadata only，不写 registry。 |
| Prediction artifact metadata | First wave | CR152 必做；metadata only，不写 prediction store。 |
| ML admission gate + CR151 status adapter | First wave | CR152 必做。 |
| Triple-barrier labeling algorithm | Deferred | CP5 默认 enforcement：first wave active selection returns `BLOCKED` unless future scope explicitly implements algorithm。 |
| Meta-label training | Deferred | Future ML wave；不得在 CR152 first wave 声称实现。 |
| Feature importance（MDI/MDA/SHAP） | Deferred | Later wave / CR154 input。 |
| Sample uniqueness weighting | Deferred | Later wave。 |
| Drift / retrain trigger | Deferred | Production monitoring / runtime CR。 |
| Model registry writer / external registry integration | Deferred and not authorized | Future runtime/storage authorization gate only。 |

### CR152 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| CR152 Story 均有 FEAT-03 / FEAT-07 / FEAT-08 归属和 `lld_policy` | PASS | 本节 `Feature 归属与 lld_policy` |
| first-wave / later-wave 边界显式化 | PASS | 本节 `CR152 First-Wave / Later-Wave 边界` |
| CP5 triple_barrier enforcement 约束显式化 | PASS | S01 说明 + Later-Wave 边界 |
| CP4 不授权实现、真实训练、真实数据、registry write 或 runtime | PASS | `process/DEVELOPMENT-PLAN-CR152.yaml#authorization_boundary` |
| CP5 前需全量设计证据确认 | PASS | `process/DEVELOPMENT-PLAN-CR152.yaml#lld_design_batch` |

### CR151 Wave A / Wave B 边界

| 范围 | 状态 | 处理 |
|---|---|---|
| Multiple testing / FDR | Wave A | CR151 必做。 |
| Robust factor statistics / Newey-West or equivalent robust t/p-value | Wave A | CR151 必做。 |
| Walk-forward / OOS fold manifest | Wave A | CR151 必做。 |
| PBO / DSR overfit risk report | Wave A | CR151 必做。 |
| StrategyAdmissionStatisticalGate aggregate status | Wave A | CR151 必做。 |
| IC decay by lag、half-life、turnover、liquidity/capacity view、orthogonalization、monotonicity、quantile spread | Deferred | CR154 / follow-up；不得被 Newey-West 字段静默视为覆盖。 |
| Regime-aware validation / regime-stratified backtest | Deferred | CR154 / follow-up；不得被 walk-forward 字段静默视为覆盖。 |
| Factor correlation clustering / redundancy de-duplication | Deferred | CR154 / follow-up；不得被 FDR 或 robust stats 静默视为覆盖。 |
| Capacity / impact、IR/TE/Active Share、PIT universe audit | Deferred | 生产级评价工件，默认不进入 CR151。 |

### CR151 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| CR151 Story 均有 FEAT-03 归属和 `lld_policy` | PASS | 本节 `Feature 归属与 lld_policy` |
| Wave A / Wave B 边界显式化 | PASS | 本节 `CR151 Wave A / Wave B 边界` |
| CP4 不授权实现或 runtime | PASS | `process/DEVELOPMENT-PLAN-CR151.yaml#authorization_boundary` |
| CP5 前需全量设计证据确认 | PASS | `process/DEVELOPMENT-PLAN-CR151.yaml#lld_design_batch` |

## 豁免与 N/A 说明

| Feature ID | 豁免 / N/A 原因 | 影响范围 | 风险接受 | 重访条件 | 责任方 |
|---|---|---|---|---|---|
| FEAT-01 | 基线已 verified，本轮只补索引 | STORY-001..013 和已关闭基础回测能力 | accepted | 修改核心 engine / strategies / reports schema 时 | meta-po / meta-se |
| FEAT-08 | 普通文档刷新可由 CR / CP8 处理 | README、USER-MANUAL、runbook | accepted | 文档新增真实操作步骤或授权语义时 | meta-doc / meta-qa |

## CR138 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| FEAT-11 / FEAT-12 已纳入矩阵 | PASS | §Feature 设计矩阵 |
| required Feature 三件套已生成 | PASS | `docs/features/runner-control-plane/*`、`docs/features/qmt-gateway-service-layer/*`；FEAT-07 / FEAT-06 复用既有三件套 |
| CR138 8 个 Story 均有 feature_design_refs 与 lld_policy | PASS | §Story 下游消费表 |
| CP5 前不授权 runtime | PASS | DQ-FD-CR138-03 |

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 所有 Feature / Epic 均已判定 | PASS | §Feature 设计矩阵覆盖 FEAT-01..10，并新增 FEAT-10-CR053 scoped migration dry-run |
| required Feature 均有产物计划或已生成 | PASS | FEAT-02/03/04/05/06/07 已有索引；FEAT-09、FEAT-10 与 FEAT-10-CR053 已生成 DESIGN / TEST-PLAN / TASKS |
| 每个 Story 均有 feature_design_refs 与 lld_policy | PASS | CR046-S01..S07、CR051-S01..S06、CR053-S01..S05 均已在 Story 下游消费表登记 |
| 提前确认的关键决策已进入人工决策队列或 N/A | PASS | DQ-FD-001..002 已在 CR-031 留痕；DQ-FD-CR046-01 与 DQ-FD-CR051-01 已作为 CP4 设计决策记录，CP5 前无新增人工阻断项 |

## CR-139 增量：策略生产数据底座 Feature 归属与 lld_policy

> 来源：CR-139 companion HLD「Strategy Data Foundation」+ handoff v0.7 §3 整改 45 项 + REQ-201..249。Feature 归属按写侧/读侧分离（AGA-1 A1，CP3 已确认）：多数读侧项归 FEAT-02 读侧子域，ML feature 层归 FEAT-03，交易审计链归 FEAT-06/11，配置层归 FEAT-02/12/14/03。跨边界项 T6 run-id 贯通：owner = FEAT-02 写侧（run-id 生成），消费方 = FEAT-11（RunEvidenceIndex）+ FEAT-06（broker event）。

### lld_policy 判定规则（CR-139 专用）

| 整改类别 | lld_policy.required_level | trigger_reasons | rationale |
|---|---|---|---|
| (a) 已设计未实现（C1/R3/R4/V1/M2/T1，6 项） | technical-note | 消费既有 HLD-DATA-LAKE 契约 | HLD 已有契约，LLD 消费即可，不刷新设计（handoff §4.2） |
| (b) 设计过期/缺失（N2/N3/C2/C3/C4/M1/M3，7 项） | full-lld（C2/N1 类结构性）/ technical-note（命名/整理类） | 设计需刷新 | 命名/整理类低风险 technical-note；C2 去重结构性 full-lld |
| (c) 范围扩展（N1/R1/R2/V2/V3/V4/M4/E1/E2/T4/T2/T3，12 项） | full-lld | cross-module-contract / data-model / external-interface | 范围扩展需完整 LLD |
| (d1) 遗漏分析纯新建（L3/E4/T7/T8/X1/X2，6 项） | full-lld | 新建对象 / 无既有合同 | 纯新建必须 full-lld |
| (d2) 既有合同闭环（L1/L2/L4/L5/E3/E5/T5/T6/F1/F2/F3/F4/X3/X4，14 项） | technical-note | 既有合同接通闭环 / 版本化扩展 / 前置门禁 | 不重复设计（REQ-249），走集成闭环 LLD |

### Feature 归属表（REQ-201..245 → Feature → lld_policy）

| REQ / 整改 ID | 整改项 | 类别 | Owner Feature | lld_policy.required_level | 既有合同位置（d2/c 闭环） |
|---|---|---|---|---|---|
| REQ-201 | N1 run_id 分区键治理 | c | FEAT-02 写侧 | full-lld | — |
| REQ-202 | N2 run_id 前缀规约 + unknown 修复 | b | FEAT-02 写侧 | technical-note | — |
| REQ-203 | N3 CR 编号不烧进路径 | b | FEAT-02 写侧 | technical-note | — |
| REQ-204 | C1 PIT as-of reader（P0） | a | FEAT-02 读侧 | technical-note | HLD-DATA-LAKE §17.7.1/§14 |
| REQ-205 | C2 分区去重（P0，C2a 画像 + C2b 读层去重） | b | FEAT-02 读侧 | full-lld | — |
| REQ-206 | C3 events schema 修复 | b | FEAT-02 写侧 | technical-note | — |
| REQ-207 | C4 写入去重保证 | b | FEAT-02 写侧 | technical-note | — |
| REQ-208 | M1 catalog/manifest 定主 | b | FEAT-02 写侧 | technical-note | — |
| REQ-209 | M2 lineage_checksum 回填 | a | FEAT-02 写侧 | technical-note | HLD-DATA-LAKE §17.7.1 |
| REQ-210 | M3 quality/ 分区整理 | b | FEAT-02 写侧 | technical-note | — |
| REQ-211 | M4 CR→数据审计链 | c | FEAT-02 写侧 | technical-note | — |
| REQ-212 | T7 整改回归基线 + 黄金值（d1） | d1 | FEAT-02 | full-lld | — |
| REQ-213 | T8 整改对象清册（d1） | d1 | FEAT-02 | full-lld | — |
| REQ-214 | R1 panel reader（P0） | c | FEAT-02 读侧（FEAT-03 消费） | full-lld | 复用 `readers.py:2728 read_dataset` published 门禁 |
| REQ-215 | R2 ML 接入 lake 废除旁路 | c | FEAT-03 | full-lld | — |
| REQ-216 | R3 DuckDB 只读 adapter（D4） | a | FEAT-02 读侧 | technical-note | HLD-DATA-LAKE §17.6/§17.7 |
| REQ-217 | R4 列裁剪/谓词下推 | a | FEAT-02 读侧 | technical-note | HLD-DATA-LAKE §17.6 |
| REQ-218 | V2 训练快照概念 | c | FEAT-03 | full-lld | — |
| REQ-219 | V3 feature/label/artifact 层（D5） | c | FEAT-03 | full-lld | — |
| REQ-220 | V4 schema 演进 + 实盘契约冻结（HIGH3） | c | FEAT-02/03 | full-lld | — |
| REQ-221 | E1 ExperimentManifest 闭环 | c | FEAT-03/11 | full-lld | `engine/research_manifest.py:152` |
| REQ-222 | E2 模型 artifact hash 闭环 | c | FEAT-03 | full-lld | `engine/strategy_admission_package.py:127` |
| REQ-223 | E3 label 泄漏统一 gate（d2） | d2 | FEAT-03 | technical-note | `factor_model_validation.py:376/561`、`factor_robustness.py:53` |
| REQ-224 | E4 离线/在线一致性（d1） | d1 | FEAT-03 | full-lld | — |
| REQ-225 | E5 split manifest 冻结（d2） | d2 | FEAT-03 | technical-note | 既有 embargo/split 策略 |
| REQ-226 | T2 PIT 正确性回归测试 | c | FEAT-02 读侧 | full-lld | — |
| REQ-227 | T3 去重正确性测试 | c | FEAT-02 读侧 | full-lld | — |
| REQ-228 | T1 DuckDB 只读 e2e 测试 | a | FEAT-02 读侧 | technical-note | `tests/test_cr014_duckdb_*.py` |
| REQ-229 | X1 复权因子 PIT 校验（d1） | d1 | FEAT-02 写侧 | full-lld | — |
| REQ-230 | X2 跨源交易日历/时区一致性（d1） | d1 | FEAT-02 写侧 | full-lld | — |
| REQ-231 | X4 PIT universe 成分链（d2） | d2 | FEAT-14 | technical-note | `engine/contracts.py:270` |
| REQ-232 | V1 published pointer / read selector（P0） | a | FEAT-02 写侧/读侧 | technical-note | HLD-DATA-LAKE §5/§17.4、`publish.py:605` |
| REQ-233 | L3 读审计 log + run-id 贯通（d1） | d1 | FEAT-02 读侧（+FEAT-11 消费） | full-lld | — |
| REQ-234 | L4 readiness 读前门禁（d2） | d2 | FEAT-02 读侧（+FEAT-14 消费） | technical-note | `market_data/readiness.py:462` |
| REQ-235 | X3 decision_time lookahead 阻断（d2） | d2 | FEAT-02 读侧 | technical-note | `market_data/readers.py:227` |
| REQ-236 | F1 版本化 benchmark + risk-free curve（d2） | d2 | FEAT-02 | technical-note | `market_data/benchmarks.py:99/114` |
| REQ-237 | F2 版本化 commission/费用/滑点（d2） | d2 | FEAT-12 | technical-note | `trading/qmt_gateway_contracts.py:997` |
| REQ-238 | F3 版本化 universe/risk policy（d2） | d2 | FEAT-14 | technical-note | `engine/mature_multifactor_framework.py:228` |
| REQ-239 | F4 版本化政策周期/shortability（d2，P2） | d2 | FEAT-03 | technical-note | `engine/factor_model_validation.py:444`、`config/policy_cycles.yaml` |
| REQ-240 | T4 BrokerLakeSchema 闭环实盘写 + 审计链 | c | FEAT-06 | full-lld | `trading/broker_lake.py` schema registry |
| REQ-241 | T5 CommissionSchedule 前置成本门禁（d2） | d2 | FEAT-12 | technical-note | `trading/qmt_gateway_contracts.py:997` |
| REQ-242 | T6 数据 run-id 贯穿 RunEvidenceIndex/broker event（d2，跨边界） | d2 | FEAT-02 写侧（生成）+ FEAT-11/06（消费） | technical-note | `trading/strategy_runner/evidence_index.py:19` |
| REQ-243 | L1 增量刷新接通日级 append 执行（d2） | d2 | FEAT-02 写侧 | technical-note | `market_data/incremental.py:248` |
| REQ-244 | L2 published pointer 接通真实前移（d2） | d2 | FEAT-02 写侧 | technical-note | `market_data/publish.py:605` |
| REQ-245 | L5 replay 接通 published as_of 重放（d2，P2） | d2 | FEAT-02 读侧 | technical-note | `market_data/replay.py:215`、`cli.py cmd_p0_replay` |

### 跨边界项归属

| 整改 ID | 跨边界范围 | owner | 消费方 | 说明 |
|---|---|---|---|---|
| T6 (REQ-242) | FEAT-02/03/06/11 | FEAT-02 写侧（run-id 生成） | FEAT-11 RunEvidenceIndex、FEAT-06 broker event | run-id 单向贯通，消费方只读，不得反向修改 |
| R1 (REQ-214) | FEAT-02/03 | FEAT-02 读侧 | FEAT-03 ML | panel reader owner 在读侧，FEAT-03 只读消费 |
| L3 (REQ-233) | FEAT-02/11 | FEAT-02 读侧（读审计 log） | FEAT-11 RunEvidenceIndex | 读审计 log owner 在读侧，FEAT-11 run-id 关联 |
| L4 (REQ-234) | FEAT-02/14 | FEAT-02 读侧（门禁） | FEAT-14 模拟盘消费 | readiness gate owner 在读侧 |
| F1-F4 (REQ-236..239) | FEAT-02/12/14/03 | FEAT-02（config_facts release） | FEAT-12/14/03 配置消费 | release 闭环复用 V1 pointer 语义（AGA-5 E1） |

### CR-139 关键决策（AGA 推荐方案，CP3 已确认 A1/C1/E1）

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 | 状态 |
|---|---|---|---|---|---|---|---|---|
| DQ-FD-CR139-01 (AGA-1) | architecture | 写侧/读侧分离是否分层同属 FEAT-02 | A1 分层同属 FEAT-02 + HLD 分读写章 | A3 拆 FEAT-02R | A1 不增 Feature、可回退；A3 边界最清但增复杂度 | Feature 边界、Story owner | CP5 读写冲突 → 升级 A3 | confirmed-cp3 (A1/C1/E1, CP3 approved 2026-06-28T17:30:00+08:00) |
| DQ-FD-CR139-02 (AGA-3) | architecture | ML feature 层归属与切换条件 | C1 lake features/ 子层 + DEF-139-01 | C2 独立 feature store（deferred） | C1 不引入依赖；C2 在线 serving 友好但增依赖 | V3 物理归属、依赖 | feature 规模/在线 serving → 切换 C2 | confirmed-cp3 (A1/C1/E1, CP3 approved 2026-06-28T17:30:00+08:00) |
| DQ-FD-CR139-03 (AGA-5) | architecture | 配置类事实源版本化机制 | E1 复用 V1 pointer 语义 + config_facts 子目录 | E2 各类独立 registry | E1 机制统一；E2 量身定制但四套机制 | F1-F4 release 闭环 | policy_cycle 语义差异过大 → 该类独立 | confirmed-cp3 (A1/C1/E1, CP3 approved 2026-06-28T17:30:00+08:00) |

### 重访条件增量

| Feature | 重访条件（增量） |
|---|---|
| FEAT-02 | 新增 dataset/publish 规则、DuckDB 事实边界、读写子域文件所有权冲突、readiness 门禁语义、config_facts release 机制、schema 演进规则、Wave1 基线门顺序变化时 |
| FEAT-03 | ML feature 层切换独立 store、feature schema 变化、旁路废除回归、policy_cycle 版本化语义、ExperimentManifest/split manifest 字段变化时 |
| FEAT-06 | broker lake 实盘写授权、run-id 贯通 schema 变化、审计链字段变化时 |
| FEAT-11 | RunEvidenceIndex run-id 贯通 schema、读审计 log 关联字段变化时 |
| FEAT-12 | CommissionSchedule 版本化字段、成本门禁前置语义变化时 |
| FEAT-14 | universe/risk policy 版本化字段、PIT universe 成分链构建规则变化时 |

### CR-139 矩阵自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| REQ-201..245 全部有 Feature 归属 | PASS | §Feature 归属表（机械核对 45 项） |
| 每个 REQ 有 lld_policy | PASS | §lld_policy 判定规则 + 归属表 |
| d2 14 项标注既有合同位置（闭环非新建） | PASS | 归属表"既有合同位置"列 + §0 核验 |
| 跨边界项 T6/R1/L3/L4/F1-F4 归属明确 | PASS | §跨边界项归属 |
| AGA 推荐方案 CP3 已确认 A1/C1/E1 | PASS | DQ-FD-CR139-01..03 confirmed-cp3 |
| CP3 前不授权 runtime / 物理分区迁移 | PASS | REQ-247/248、Wave1 N1 后置 |
