---
status: "cr172-path-i-cp5-r3-minimal-correction-ready"
version: "1.30"
source_blueprint: "docs/design/BLUEPRINT.md"
source_hld:
  - "docs/design/HLD.md"
  - "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
  - "process/archive/design-cr-docs/HLD-WALK-FORWARD-OOS-EVIDENCE.md"
  - "process/archive/design-cr-docs/HLD-ECONOMIC-COST-IMPACT-EVIDENCE.md"
  - "docs/design/HLD-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
  - "process/archive/design-cr-docs/HLD-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md"
  - "docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md"
  - "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_adr:
  - "docs/design/ARCHITECTURE-DECISION.md"
  - "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
  - "process/archive/design-cr-docs/ARCHITECTURE-DECISION-WALK-FORWARD-OOS-EVIDENCE.md"
  - "process/archive/design-cr-docs/ARCHITECTURE-DECISION-ECONOMIC-COST-IMPACT-EVIDENCE.md"
  - "docs/design/ARCHITECTURE-DECISION-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
  - "process/archive/design-cr-docs/ARCHITECTURE-DECISION-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md"
  - "docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md"
  - "docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
change: "CR-172"
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
| 1.16 | 2026-07-02 | host-orchestrator | CR153 CP3 approved 后增补 Event-Driven Strategy E2E Framework CP4：FEAT-03 下新增事件研究 first-wave Story 消费、`lld_policy`、DAG/Wave、EV-GAP-7 multiple-testing slot、CR154 deferred 边界和 no-feed/no-runtime/no-store 边界。 |
| 1.17 | 2026-07-03 | host-orchestrator | CR154 CP3 approved 后增补 Cross-Strategy Production Reliability Gates CP4：新增 FEAT-15 cross-strategy reliability gates 三件套、8 个 Story、Gate 5 显式 Story、Phase A runnable fixture schema 归属、Gate 6 tier resolver full-lld 和 CP4/CP5 follow-through acceptance hooks。 |
| 1.18 | 2026-07-04 | host-orchestrator | CR155 CP3 approved 后增补 Daily Multifactor Baseline Strategy Artifact CP4：新增 FEAT-16 baseline artifact 三件套、5 个 Story、readonly provenance、historical/OOS validation、admission composition、rerun consistency 和 no-runtime/no-write CP5 attention hooks。 |
| 1.19 | 2026-07-05 | host-orchestrator | CR157 CP3 approved 后增补 Stage 2 Multifactor Research Framework Upgrade CP4：新增 FEAT-17 mature admission package、FEAT-18 research evidence traceability、FEAT-19 Stage 2/3 handoff guardrails 三件套，5 个 Story、refs-only evidence、fail-closed handoff、no-runtime guard 和 event/ML adapter deferred 边界。 |
| 1.20 | 2026-07-05 | host-orchestrator | CR158 CP3 approved 后增补 Event + ML Strategy Adapter Unified Implementation CP4：复用 FEAT-03 / FEAT-07 / FEAT-08，新增 6 个 Story、thin shared core、typed event/ML extension、refs-only evidence、no-runtime counters 和 release wording boundary。 |
| 1.21 | 2026-07-11 | meta-se-critical | CR163 CP3 approved 后增补 FEAT-20 experiment-family lineage core、FEAT-21 producer adapters、FEAT-22 admission projection 三套 required Feature 设计；FEAT-23 standalone validation waived，由三套 TEST-PLAN 汇总覆盖；五 Story 全部 full-lld。 |
| 1.22 | 2026-07-13 | host-orchestrator inline meta-se | CR166 CP3 approved 后增补 FEAT-166-01..05 五套 required Feature 设计；五 Story 全部 full-lld、五个串行安全 Wave，保持 fixture/static 与 no-runtime 边界。 |
| 1.23 | 2026-07-14 | host-orchestrator inline meta-se | CR168 CP3 revised/approved 后增补 FEAT-168-01..03，并增量复用 FEAT-166-01；五 Story 全部 full-lld、五个串行安全 Wave，固化 component/envelope identity 分域与 Gate4 adapter-local containment。 |
| 1.24 | 2026-07-14 | host-orchestrator inline fallback | CR168 专题 CP3 正文归档后更新其 HLD/ADR source path；Feature 判定、Story、lld_policy 和实施语义不变。 |
| 1.25 | 2026-07-14 | host-orchestrator inline meta-se | CR169 CP3 approved 后增补 FEAT-169-01..04、12/12 三件套、5 个串行 Story、13 字段 correlation、strict 7-key joint adapter 与 Stage2 7/7 exit guard。 |
| 1.26 | 2026-07-15 | host-orchestrator inline meta-se | CR170 CP3 approved 后增量复用 FEAT-15；新增 4 个 full-lld Story、21-unit 五态与 15/5/1 方向清单、受保护 merge、T0-T3 admission 及 caller/T3/adapter 回归义务。 |
| 1.27 | 2026-07-16 | meta-se-critical | CR173 CP3 approved 后新增 FEAT-173-01 estimator-only Feature 三件套、3 个 full-lld Story、3 个串行 Wave；冻结 exact-rational spectral estimator、七字段 evidence、6×3 golden/授权守卫，public C1 projection Story=0。 |
| 1.28 | 2026-07-18 | meta-se-critical | CR172 PATH-I CP3 approved 后新增 3 个 required Feature、5 个 full-lld Story 与 5 个串行 Wave；初始 producer feasibility 口径已由 v1.29 的 source audit 安全收窄。 |
| 1.29 | 2026-07-18 | meta-se-critical | CR172 CP5 R1 安全收窄：I01/S02 改为 pure contract + fixture producer port，现有 runner/lineage diff=`0/0`；forward-label proxy 禁入；fixture/real binding 与 verified seal digest 上移为 public contract；真实 producer/原子 lineage 转独立前置 CR。 |
| 1.30 | 2026-07-18 | meta-se-critical | CR172 CP5 R2 最小整改：approved-ledger current-v1 双 false；S04 允许 S02 verifier-library dependency 但必须从 S03 selection 取数；evidence_kind 第二真相=0；REQ-013=contract-ready/runtime-enforcement-deferred。 |

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
| CP4 不授权实现、真实训练、真实数据、registry write 或 runtime | PASS | `process/DEVELOPMENT-PLAN-CR152-ML-STRATEGY-E2E-FRAMEWORK.yaml#authorization_boundary` |
| CP5 前需全量设计证据确认 | PASS | `process/DEVELOPMENT-PLAN-CR152-ML-STRATEGY-E2E-FRAMEWORK.yaml#lld_design_batch` |

## CR153 CP4 增量：Event-Driven Strategy E2E Framework

> 来源：`process/docs/design/HLD-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md`、`process/docs/design/ARCHITECTURE-DECISION-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md`、CP3 用户批准。CR153 归属 FEAT-03「研究数据集与多因子研究闭环」的 event-driven strategy first-wave foundation，不新增真实事件 feed、live listener、真实数据验证、event store/catalog/model registry write、production catalog pointer 或 runtime 授权。

### Feature 归属与 lld_policy

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 | 说明 |
|---|---|---|---|---|---|---|
| CR153-S01-event-research-time-pit-contracts | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | EventResearchSpec、三时间语义、EventRevisionPITGate、PIT/revision fail-closed | Story LLD | CP5 必须明确 `event_available_at > decision_time` 的 `BLOCKED` 行为；不得从 occurred/announced 推断 available。 |
| CR153-S02-event-study-method-test-slots | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | estimation window、event window、normal return model、test family、EV-GAP-7 multiple-testing/data-snooping slot | Story LLD | `multiple_testing_or_data_snooping_slot` 必须包含 `family_id`、`tested_window_count`、`correction_method`、`adjusted_p_value`、`status`、`report_ref` 或 `n/a_reason`；算法仍 slot-only。CP5 LLD 必须声明 S02 只拥有 method/test family/multiple-testing 字段，不拥有 S03 bias/CV/universe 字段。 |
| CR153-S03-event-bias-risk-audit-slots | FEAT-03 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | overlap / cluster / endogeneity、event CV split audit、universe PIT audit、CR154 deferred refs | Story LLD | CR153 只做 machine-visible slot/status/ref/n/a reason；完整 CV/survivorship/capacity/impact/regime/reconciliation 归 CR154。CP5 LLD 必须声明 S03 只拥有 bias/CV/universe PIT audit 字段，不重定义 S02 method/test family/multiple-testing 字段。 |
| CR153-S04-event-admission-gate-adapter | FEAT-03 / FEAT-07 safety boundary | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | EventStrategyAdmissionGate、CR151/CR152 四态 status adapter、StrategyAdmissionPackage linkage、forbidden operation counters | Story LLD | Event gate PASS 不等于 feed/runtime/paper/live/broker/trading readiness。 |
| CR153-S05-event-trace-evidence-wording | FEAT-03 / FEAT-08 | `docs/features/factor-research-loop/TEST-PLAN.md`、`TASKS.md` | technical-note | event -> signal -> target/order-intent trace metadata、CP6/CP7/CP8 evidence wording、no-runtime/no-order boundary | Story technical note | 只保留 trace refs 和 fixture/static wording；不得创建真实 order flow、event store、catalog 或 registry 写入。CP5 technical note 必须枚举精确 CR153 artifact target 或逐项 N/A，不得保留宽泛 release/docs owner。 |

### CR153 First-Wave / Later-Wave 边界

| 范围 | 状态 | 处理 |
|---|---|---|
| EventResearchSpec and three-time semantics | First wave | CR153 必做。 |
| EventRevisionPITGate | First wave | CR153 必做；available-after-decision 必须 fail closed。 |
| EventStudyMethodSpec | First wave | CR153 必做；estimation/event window 和 normal return model 必须结构化。 |
| EventStudyTestReport slots | First wave | CR153 必做；test family / sample / p-value / adjusted p-value / refs 可机器读取。 |
| Multiple testing / data snooping slot | First wave | CR153 必做；EV-GAP-7 明确进入 S02，不实现 White/Hansen/PBO/DSR 等算法。 |
| Overlap / cluster / endogeneity slots | First wave | CR153 必做；status/refs/n/a reason，不实现完整校正算法。 |
| Event CV and universe PIT audit slots | First wave slot only | CR153 保留 slot 和 split/audit refs；完整治理归 CR154。 |
| EventStrategyAdmissionGate and adapter | First wave | CR153 必做；复用四态 admission 语义。 |
| Event-to-signal/order-intent trace | First wave metadata only | CR153 只做 trace refs，不启动 runtime/order flow。 |
| Full event statistics algorithms | Deferred | Patell / BMP / bootstrap / cluster robust / IV / PSM / matching 等后续 CR。 |
| Live event feed / listener / paper OMS / broker adapter | Deferred and not authorized | Future runtime/storage authorization gate only。 |
| Capacity / impact / regime / reconciliation | Deferred | CR154 或后续正式 CR。 |

### CR153 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| CR153 Story 均有 FEAT-03 / FEAT-07 / FEAT-08 归属和 `lld_policy` | PASS | 本节 `Feature 归属与 lld_policy` |
| 原始 EV-GAP-1..9 编号未被重解释 | PASS | HLD §11；本节保留 CR153 专属 Story，不改 EV-GAP 编号 |
| EV-GAP-7 multiple testing / data snooping slot 显式化 | PASS | S02 说明 + First-Wave 边界 |
| CP4 不授权实现、真实 feed/listener、真实数据、event store/catalog/model registry write 或 runtime | PASS | `process/DEVELOPMENT-PLAN-CR153-EVENT-DRIVEN-STRATEGY-E2E.yaml#authorization_boundary` |
| CP5 前需全量设计证据确认 | PASS | `process/DEVELOPMENT-PLAN-CR153-EVENT-DRIVEN-STRATEGY-E2E.yaml#lld_design_batch` |

## CR154 CP4 增量：Cross-Strategy Production Reliability Gates

> 来源：`process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md`、`process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md`、CP3 用户批准和 CP4 Story split review。CR154 归属新增 FEAT-15「Cross-Strategy Production Reliability Gates」，保持 local/static/fixture-only，不新增真实 lake/NAS/provider/QMT/runtime/simulation/paper/live/trading/broker/credential/feed/order/reconciliation/store/catalog/registry/publish 授权。

### Feature 归属与 lld_policy

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 | 说明 |
|---|---|---|---|---|---|---|
| CR154-S01-shared-gate-contract-fixture-skeleton | FEAT-15 | `docs/features/cross-strategy-reliability-gates/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | shared contract、data model、status enum、first runnable fixture schema | Story LLD | Phase A 必须自带 fixture schema 和第一个可运行 fixture，避免 Phase A/B 边界空壳化。 |
| CR154-S02-statistical-artifacts-and-trap-severity | FEAT-15 | `docs/features/cross-strategy-reliability-gates/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | Gate 1、12 artifact slots、WRC/SPA severity、PBO/DSR、adapter mapping | Story LLD | B1 是关键路径；CP5 LLD 必须按 multifactor / ML / event-driven 拆出 adapter 子任务，并覆盖 FT-CR154-CP5-001/002/003。 |
| CR154-S03-cross-strategy-cv-governance | FEAT-15 | `docs/features/cross-strategy-reliability-gates/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | Gate 2、walk-forward / OOS / purged-embargo governance | Story LLD | 共享 contract + strategy-specific adapter；不得把 CR152 ML CV 语义直接套给多因子和事件策略。 |
| CR154-S04-pit-universe-survivorship-gate | FEAT-15 | `docs/features/cross-strategy-reliability-gates/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | Gate 3、survivorship-free universe、CR153 `universe_pit_audit` lifecycle | Story LLD | CR153 slot first wave 保留为 delegated source；CR154 拥有 shared PIT release-blocking wording，不构建真实 universe 数据。 |
| CR154-S05-capacity-impact-liquidity-contract | FEAT-15 | `docs/features/cross-strategy-reliability-gates/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | Gate 4、ADV participation、capacity dollars、impact enum、cost underestimation | Story LLD | `impact_model_family` 受控枚举至少含 `square_root` / `almgren_chriss` / `gatheral` / `custom` / `n/a-with-reason`；不做真实 TCA。 |
| CR154-S06-regime-attribution-reconciliation-slots | FEAT-15 | `docs/features/cross-strategy-reliability-gates/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | Gate 5、slots/status/refs/n/a、no-runtime reconciliation boundary | Story LLD | 回应 CP4 review：Gate 5 必须有显式 Story，不得隐含在 Phase D。 |
| CR154-S07-admission-default-policy-tier-resolution | FEAT-15 / FEAT-07 safety boundary | `docs/features/cross-strategy-reliability-gates/DESIGN.md`、`TEST-PLAN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | Gate 6、tier resolver、release wording、unknown profile fail-closed | Story LLD | 回应 CP4 review：tier resolution 是独立 full-lld Story；需定义 config/function 边界、fallback 和 per-strategy override 规则。 |
| CR154-S08-compatibility-follow-through-wording | FEAT-15 / FEAT-08 | `docs/features/cross-strategy-reliability-gates/TEST-PLAN.md`、`TASKS.md` | technical-note | CR151/152/153 compatibility、follow-through hooks、static evidence wording | Story technical note | 保存 MF-GAP-2/6/7 deferred、REQ anchor preservation、ML-only n/a policy、no-runtime/no-real-data release wording。 |

### CR154 First-Wave / Later-Wave 边界

| 范围 | 状态 | 处理 |
|---|---|---|
| Shared reliability gate summary and artifact refs | First wave | S01 必做。 |
| First runnable fixture schema | First wave | S01 必做；Gate-specific fixtures由 S02-S07 扩展。 |
| Gate 1 statistical artifact model | First wave | S02 必做；必须显式化 multiple-testing / FDR-BH / WRC-SPA / PBO-CSCV / DSR-deflation / trials / OOS / purge-embargo / survivorship / impact-capacity refs。 |
| Gate 2 CV governance | First wave | S03 必做。 |
| Gate 3 PIT universe / survivorship gate | First wave | S04 必做；不读取真实 universe。 |
| Gate 4 capacity / impact / liquidity contract | First wave | S05 必做；不做真实 TCA。 |
| Gate 5 regime / attribution / reconciliation slots | First wave | S06 必做；slot/status/ref/n/a only。 |
| Gate 6 admission default policy | First wave | S07 必做；unknown release profile fail-closed。 |
| CR151/152/153 compatibility and wording | First wave | S08 必做；technical-note。 |
| Exact PBO / DSR / capacity thresholds | Required-later in CP5 | S02/S05/S07 必须提供默认、config owner 或 `n/a-with-reason` policy；不做真实 calibration。 |
| MF-GAP-2/6/7 | Deferred | S08 必须声明 deferred to factor-evaluation follow-up CR。 |
| Real lake / NAS / provider / runtime / broker / feed / reconciliation / publish | Deferred and not authorized | Future runtime/data authorization gate only。 |

### CR154 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 8 个 CR154 Story 均有 FEAT-15 归属和 `lld_policy` | PASS | 本节 `Feature 归属与 lld_policy` |
| Gate 5 显式 Story | PASS | `CR154-S06-regime-attribution-reconciliation-slots` |
| Phase A fixture schema 归属明确 | PASS | `CR154-S01-shared-gate-contract-fixture-skeleton` acceptance criteria |
| Gate 6 tier resolution 独立 full-lld | PASS | `CR154-S07-admission-default-policy-tier-resolution` |
| B1 工作量作为关键路径显式化 | PASS | S02 说明 + `process/DEVELOPMENT-PLAN-CR154-CROSS-STRATEGY-RELIABILITY-GATES.yaml#cp5_attention_items` |
| CP4 不授权 LLD、实现、测试实现、真实数据或 runtime | PASS | `process/DEVELOPMENT-PLAN-CR154-CROSS-STRATEGY-RELIABILITY-GATES.yaml#authorization_boundary` |
| CP5 前需全量设计证据确认 | PASS | `process/DEVELOPMENT-PLAN-CR154-CROSS-STRATEGY-RELIABILITY-GATES.yaml#lld_design_batch` |

## CR155 CP4 增量：Daily Multifactor Baseline Strategy Artifact

> 来源：`docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md`、`docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md`、CP3 用户批准。CR155 归属新增 FEAT-16「Daily Multifactor Baseline Strategy Artifact」，允许 CR155-scoped local governed lake/current truth readonly 作为设计输入和后续验证输入；仍不授权 lake write、NAS/provider/credential/runtime/trading/broker/catalog/store/registry/publish。

### Feature 归属与 lld_policy

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 | 说明 |
|---|---|---|---|---|---|---|
| CR155-S01-baseline-artifact-contract | FEAT-16 / FEAT-03 | `docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | 新增 strategy artifact contract、identity、policy refs、claim boundary | Story LLD | 定义 `strategy_id`、universe、factor spec、signal、portfolio policy、rebalance/cost policy、evidence refs 和 non-optimal baseline 声明。 |
| CR155-S02-readonly-data-provenance-adapter | FEAT-16 / FEAT-02 / FEAT-07 | `docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md`、`TEST-PLAN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | CP2-approved readonly local governed lake/current truth boundary、provenance、forbidden operation counters | Story LLD | 只读读取合同必须证明 no lake write、no catalog mutation、no credential/env/NAS/provider/runtime。 |
| CR155-S03-backtest-oos-walkforward-validation | FEAT-16 / FEAT-03 | `docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md`、`TEST-PLAN.md` | full-lld | historical backtest、OOS/walk-forward、split manifest、cost/risk attribution refs | Story LLD | 消费 CR148 foundation；不得新增 optimizer、external framework 或 production runtime。 |
| CR155-S04-admission-gate-composition-package | FEAT-16 / FEAT-03 / FEAT-15 | `docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | CR151 statistical gate、CR154 reliability gate、admission package、paper_candidate reason | Story LLD | 保留 statistical / reliability / final package 三层状态，不能把原因折叠成单一 PASS/FAIL。 |
| CR155-S05-rerun-consistency-release-evidence | FEAT-16 / FEAT-08 | `docs/features/daily-multifactor-baseline-strategy-artifact/TEST-PLAN.md`、`TASKS.md` | full-lld | two rerun comparison、metric tolerance、CP6/CP7 evidence index、release wording | Story LLD | 两次 rerun 核心 metrics 一致是一级验收；release wording 必须说明 research artifact only。 |

### CR155 First-Wave / Later-Wave 边界

| 范围 | 状态 | 处理 |
|---|---|---|
| Standalone daily multifactor baseline artifact contract | First wave | S01 必做。 |
| CR155-scoped readonly provenance adapter | First wave | S02 必做；真实读取只能在后续批准的实现/验证阶段按只读边界执行。 |
| Historical backtest and OOS/walk-forward validation flow | First wave | S03 必做。 |
| Statistical + reliability + admission package composition | First wave | S04 必做。 |
| Two-run metric consistency and release evidence wording | First wave | S05 必做。 |
| Factor optimizer / model search / strategy discovery | Deferred | Future research CR only。 |
| Paper/live readiness or trading integration | Deferred and not authorized | Future runtime/trading authorization CR only。 |
| Lake write / catalog pointer / store or registry writes | Deferred and not authorized | Future data authorization CR only。 |

### CR155 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 5 个 CR155 Story 均有 FEAT-16 归属和 `lld_policy` | PASS | 本节 `Feature 归属与 lld_policy` |
| readonly provenance 独立 Story | PASS | `CR155-S02-readonly-data-provenance-adapter` |
| historical/OOS validation 独立 Story | PASS | `CR155-S03-backtest-oos-walkforward-validation` |
| admission composition 独立 Story | PASS | `CR155-S04-admission-gate-composition-package` |
| rerun consistency 是一级 Story | PASS | `CR155-S05-rerun-consistency-release-evidence` |
| CP4 不授权 LLD 审批、实现、测试实现、lake write、runtime 或 trading | PASS | `process/DEVELOPMENT-PLAN-CR155.yaml#authorization_boundary` |
| CP5 前需全量设计证据确认 | PASS | `process/DEVELOPMENT-PLAN-CR155.yaml#lld_design_batch` |

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
| CP4 不授权实现或 runtime | PASS | `process/DEVELOPMENT-PLAN-CR151-MULTIFACTOR-STRATEGY-STATISTICAL-ADMISSION.yaml#authorization_boundary` |
| CP5 前需全量设计证据确认 | PASS | `process/DEVELOPMENT-PLAN-CR151-MULTIFACTOR-STRATEGY-STATISTICAL-ADMISSION.yaml#lld_design_batch` |

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

## CR157 CP4 增量：Stage 2 Multifactor Research Framework Upgrade

> 来源：`docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md`、`docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md`、CP3 用户批准。CR157 关闭 Stage 2 first slice 中的 mature admission package builder、research evidence traceability、Stage 2/3 handoff hardening 和 no-runtime guard，不新增真实数据湖、NAS/provider、QMT/gateway、simulation/paper/live/trading、catalog/store/registry/publish 或外部框架运行授权。

### Feature 归属与 lld_policy

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 | 说明 |
|---|---|---|---|---|---|---|
| CR157-S01-mature-admission-package-builder-contract | FEAT-17 / FEAT-03 | `docs/features/mature-admission-package/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | shared admission package builder contract、Stage 2 exit object aggregation、fail-closed missing mandatory refs、forbidden operation counters | Story LLD | Builder 必须消费既有 FactorSpec / FactorRunSpec / label / evaluation / portfolio-risk refs，不重新实现这些对象。 |
| CR157-S02-research-evidence-index-traceability | FEAT-18 / FEAT-13 | `docs/features/research-evidence-traceability/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | refs-only evidence index、stable evidence id、source object traceability、no embedded report body | Story LLD | Evidence index 只保存 ref/hash/status/owner，不复制完整报告、测试正文或 transcript。 |
| CR157-S03-stage2-stage3-handoff-hardening | FEAT-19 / FEAT-14 | `docs/features/stage-handoff-guardrails/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | Stage 2 exit checklist、Stage 3 candidate admission handoff、blocked claim propagation、fail-closed handoff | Story LLD | Stage 3 只能接收 complete / blocked-with-reason 包；不能通过缺字段默认放行。 |
| CR157-S04-no-runtime-guard-coverage | FEAT-19 / FEAT-07 | `docs/features/stage-handoff-guardrails/DESIGN.md`、`docs/features/stage-handoff-guardrails/TEST-PLAN.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | no-runtime / no-real-data / no-publish guard coverage、negative fixture matrix、operation counters | Story LLD | 所有 forbidden counters 非零必须 BLOCKED；CP4/CP5 不授权实现或运行。 |
| CR157-S05-docs-release-backlog-alignment | FEAT-18 / FEAT-19 / FEAT-08 | `docs/features/research-evidence-traceability/TASKS.md`、`docs/features/stage-handoff-guardrails/TASKS.md` | technical-note | roadmap/component docs alignment、deferred adapter refs、release wording and backlog hygiene | Story technical note | 文档收口不得声称 event adapter、ML adapter、真实数据验证或发布能力已完成。 |

### First-Slice / Deferred 边界

| 范围 | 状态 | 处理 |
|---|---|---|
| Mature admission package builder | First slice | CR157 必做；fixture/static-only，输出 package summary、mandatory ref completeness、blocked claims 和 forbidden counters。 |
| Research evidence index | First slice | CR157 必做；refs-only evidence traceability，禁止嵌入完整报告正文。 |
| Stage 2 / Stage 3 handoff hardening | First slice | CR157 必做；Stage 3 只消费 complete 或 blocked-with-reason handoff。 |
| No-runtime guard coverage | First slice | CR157 必做；覆盖 data lake/provider/QMT/runtime/trading/store/publish/external framework negative cases。 |
| Event adapter implementation | Deferred | `DF-CR157-001`；后续 event-specific CR 才能落地。 |
| ML adapter implementation | Deferred | `DF-CR157-002`；后续 ML-specific CR 才能落地。 |
| Real data validation / provider fetch / publish | Not authorized | 需要独立 runtime_authorization 或正式 CR，不由 CP4/CP5 默认放行。 |

### CP5 注意项

| Attention ID | Story | Requirement |
|---|---|---|
| CP5-FOCUS-CR157-001 | CR157-S01 | 明确 mandatory Stage 2 exit refs：FactorSpec、FactorRunSpec、factor panel、label window、evaluation、portfolio/risk、admission package、evidence index。 |
| CP5-FOCUS-CR157-002 | CR157-S01 / S02 | `missing mandatory ref`、`stale ref`、`invalid hash`、`n/a-with-reason`、`blocked claim` 的状态映射必须可机器验证。 |
| CP5-FOCUS-CR157-003 | CR157-S02 | Evidence index 必须 refs-only，禁止复制完整统计报告、完整 TEST-MATRIX、完整 REVIEW 或会话 transcript。 |
| CP5-FOCUS-CR157-004 | CR157-S03 | Stage 2 / Stage 3 handoff 必须 fail closed；Stage 3 不能把缺字段当作 PASS。 |
| CP5-FOCUS-CR157-005 | CR157-S04 | Forbidden operation counters 必须覆盖 lake/NAS/provider/QMT/runtime/trading/broker/store/catalog/registry/publish/external framework/Git remote write。 |
| CP5-FOCUS-CR157-006 | CR157-S05 | Deferred event/ML adapter 只能写 backlog/ref，不得在 release wording 中声明已实现或可运行。 |

### CR157 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| CR157 Story 均有 FEAT-17 / FEAT-18 / FEAT-19 或既有安全/文档 Feature 归属 | PASS | 本节 `Feature 归属与 lld_policy` |
| first-slice / deferred / not-authorized 边界显式化 | PASS | 本节 `First-Slice / Deferred 边界` |
| CP5 注意项已覆盖 Stage 2 exit refs、refs-only evidence、fail-closed handoff 和 no-runtime guard | PASS | 本节 `CP5 注意项` |
| CP4 不授权 LLD approval、实现、测试实现、真实数据、runtime、publish 或交易 | PASS | `process/DEVELOPMENT-PLAN-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.yaml#authorization_boundary` |

## CR158 CP4 增量：Event + ML Strategy Adapter Unified Implementation

> 来源：`docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md`、`docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md`、CP3 用户批准。CR158 复用 FEAT-03 作为策略研究 adapter 合同 owner，FEAT-07 作为 no-runtime / no-real-operation guard owner，FEAT-08 作为 release wording / docs boundary owner；不新增真实 event feed、真实 ML training、external model service、model registry、provider/lake/NAS/credential、QMT/gateway、simulation/paper/live/trading、publish 或 Git remote write 授权。

### Feature 归属与 lld_policy

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 | 说明 |
|---|---|---|---|---|---|---|
| CR158-S01-shared-adapter-core-contract | FEAT-03 / FEAT-07 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | shared adapter core schema、validation result contract、authorization flags、blocked reason semantics | Story LLD | 定义 StrategyTypeAdapterCore 最小 7 类字段和 AdapterValidationResult；event-only / ML-only 字段不得进入 core。 |
| CR158-S02-event-strategy-adapter-extension | FEAT-03 / FEAT-07 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | event source/time/payload schema/alignment policy/signal output/blocked reason refs、no-feed boundary | Story LLD | 只允许 fixture/static event refs；不得创建 live listener、provider fetch、gateway call 或 event store writer。 |
| CR158-S03-ml-strategy-adapter-extension | FEAT-03 / FEAT-07 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`、`docs/features/runtime-authorization-safety/DESIGN.md` | full-lld | training snapshot/feature set/label policy/model artifact/validation report/prediction signal refs、no-training/no-registry boundary | Story LLD | 只允许 fixture/static model artifact refs；不得训练真实模型、调用外部模型服务或写 model registry / prediction store。 |
| CR158-S04-evidence-handoff-typed-refs | FEAT-03 / FEAT-13 / FEAT-14 | `docs/features/factor-research-loop/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | evidence index typed refs-only extension、Stage 2/3 handoff refs、body-copy count 0 | Story LLD | Event / ML evidence extension 只保存 refs、hash、status、owner 和短元数据；不得复制正文、payload、模型二进制、diff 或 transcript。 |
| CR158-S05-no-runtime-guard-counters | FEAT-07 | `docs/features/runtime-authorization-safety/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | forbidden operation counter report、fail-closed guard、CP7 security validation | Story LLD | 任一 forbidden counter 非 0 必须 BLOCKED；覆盖 feed/training/provider/lake/NAS/credential/runtime/trading/registry/publish/Git remote。 |
| CR158-S06-verification-release-boundary | FEAT-08 / FEAT-07 | `docs/features/factor-research-loop/TEST-PLAN.md`、`docs/features/runtime-authorization-safety/TEST-PLAN.md`、`docs/features/runtime-authorization-safety/TASKS.md` | technical-note | TEST-MATRIX 回链、release wording、no runtime readiness claim、CP8 follow-through | Story technical note | 收口 docs、release notes、verification report wording；明确 local/static/fixture adapter readiness 不等于 production/runtime readiness。 |

### First-Slice / Deferred / Not-Authorized 边界

| 范围 | 状态 | 处理 |
|---|---|---|
| Thin shared adapter core | First slice | CR158 必做；只定义可被 event/ML 共用的 strategy type、input/output/evidence/blocked/authorization/handoff refs。 |
| Event typed extension | First slice | CR158 必做；fixture/static refs only，不接真实 event feed。 |
| ML typed extension | First slice | CR158 必做；fixture/static refs only，不训练真实模型、不写 registry。 |
| Evidence typed refs and handoff refs | First slice | CR158 必做；refs-only，不复制正文或二进制。 |
| No-runtime guard counter report | First slice | CR158 必做；所有 forbidden operation counters 必须为 0。 |
| Real event feed / live listener | Not authorized | 后续 runtime authorization CR 才能启动。 |
| Real ML training / external model service / registry promotion | Not authorized | 后续 runtime / model registry authorization CR 才能启动。 |
| Provider/lake/NAS/credential/QMT/gateway/trading/publish/Git remote write | Not authorized | 任何触达都需要独立授权。 |

### CP5 注意项

| Attention ID | Story | Requirement |
|---|---|---|
| CP5-FOCUS-CR158-001 | CR158-S01 | Shared core 字段必须保持 thin；event-only 与 ML-only 字段不得互设必填。 |
| CP5-FOCUS-CR158-002 | CR158-S02 / S03 | Event extension 与 ML extension 必须各自 fail-closed；缺 P0 refs 时不得降级为 warning。 |
| CP5-FOCUS-CR158-003 | CR158-S04 | Evidence extension 必须 refs-only，body_copy_count 必须为 0。 |
| CP5-FOCUS-CR158-004 | CR158-S05 | Forbidden counters must cover feed、training、provider、lake、NAS、credential、runtime、trading、registry、publish、external framework 和 Git remote write。 |
| CP5-FOCUS-CR158-005 | CR158-S06 | Release wording 只能声明 local/static/fixture adapter readiness；不得声明 production/runtime/trading/model-registry readiness。 |

### CR158 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| CR158 Story 均有 FEAT-03 / FEAT-07 / FEAT-08 或相邻 Feature 归属 | PASS | 本节 `Feature 归属与 lld_policy` |
| first-slice / deferred / not-authorized 边界显式化 | PASS | 本节 `First-Slice / Deferred / Not-Authorized 边界` |
| CP5 注意项覆盖 core、extension、refs-only evidence、no-runtime guard 和 release wording | PASS | 本节 `CP5 注意项` |
| CP4 不授权 LLD approval、实现、测试实现、真实 feed、真实训练、真实数据、runtime、registry、publish 或交易 | PASS | `process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml#authorization_boundary` |

## CR163 CP4 增量：Trial Lineage Instrumentation

> 来源：`docs/design/BLUEPRINT-TRIAL-LINEAGE-INSTRUMENTATION.md`、`docs/design/HLD-TRIAL-LINEAGE-INSTRUMENTATION.md`、`docs/design/ARCHITECTURE-DECISION-TRIAL-LINEAGE-INSTRUMENTATION.md` 和 CP3 用户批准。CR163 新增可审计的 experiment-family raw lineage 事实源，但不计算 effective trial count 或 C1 statistical evidence，不重建 CR155 历史，不扩大 runtime/data/credential/external-write 授权。

### Feature 适用性判定

| Feature ID | Feature / Epic | 适用性 | 判定理由 | 需要产物 | 关联 Story | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|
| FEAT-20 | Experiment-family lineage core | required | 六个持久化对象、FamilyLineageSession、状态机、幂等/冲突、canonical JSON/JSONL、immutable seal 与 supersession 被 S01/S02/S05 共享 | `docs/features/experiment-family-lineage/{DESIGN,TEST-PLAN,TASKS}.md` | CR163-S01, S02, S05 | full-lld | object/schema/state/hash/storage/supersession 或 single-writer 假设变化 |
| FEAT-21 | Trial-lineage producer adapters | required | 两条 producer chains / CPI-001..004 跨四个调用点共享 pre-search/session/trial identity 防双计数 contract | `docs/features/trial-lineage-producer-adapters/{DESIGN,TEST-PLAN,TASKS}.md` | CR163-S03, S05 | full-lld | 新增 candidate producer、wrapper/hook owner 或跨语言 command transport |
| FEAT-22 | Strategy-admission lineage projection | required | 跨 CR151/CR154/admission package 的 availability/ref/raw count/status-worsening contract；manual count reconciliation-only | `docs/features/strategy-admission-lineage-projection/{DESIGN,TEST-PLAN,TASKS}.md` | CR163-S04, S05 | full-lld | existing gate schema、availability、manual/backfill policy 或 claim tier变化 |
| FEAT-23 | Trial-lineage verification orchestration | waived-as-standalone | 不拥有新生产数据或公共接口；S05 只组合 FEAT-20..22 的 contract/fixture/security/regression 验证，独立 DESIGN 会复制三套测试职责 | none；消费三套 TEST-PLAN | CR163-S05 | full-lld（Story） | 若形成独立验证 runtime/service、共享 fixture schema 或跨 CR validation API，升级 required |

### Story 下游消费表

| Story ID | Owner Feature | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据 |
|---|---|---|---|---|---|
| CR163-S01-family-contract-validator | FEAT-20 | `docs/features/experiment-family-lineage/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | six-object public contract / state / validation / availability / security boundary | Story LLD |
| CR163-S02-recorder-seal-supersession | FEAT-20 | `docs/features/experiment-family-lineage/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | append-only storage / idempotency / canonicalization / immutable seal / recovery | Story LLD |
| CR163-S03-two-producer-chain-instrumentation | FEAT-21 / FEAT-20 | `docs/features/trial-lineage-producer-adapters/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`、`docs/features/experiment-family-lineage/DESIGN.md` | full-lld | cross-module 2 chains / 4 mappings / identity ownership / file coordination | Story LLD |
| CR163-S04-existing-admission-projection | FEAT-22 / FEAT-20 | `docs/features/strategy-admission-lineage-projection/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`、`docs/features/experiment-family-lineage/DESIGN.md` | full-lld | three consumer surfaces / compatibility / fail-closed / claim ceiling | Story LLD |
| CR163-S05-integrity-recovery-permission-regression | FEAT-20 / FEAT-21 / FEAT-22 / FEAT-23 | 三套 `DESIGN.md` / `TEST-PLAN.md` / `TASKS.md` | full-lld | cross-module fixtures / tamper / supersession / permission / CR155 regression | Story LLD；FEAT-23 standalone waiver 作为附加依据 |

### CP5 注意项

| Attention ID | Story | Requirement |
|---|---|---|
| CP5-FOCUS-CR163-001 | S01 | 六对象与 façade 字段、state transitions、machine blocked codes、validation target ref/hash 必须完整。 |
| CP5-FOCUS-CR163-002 | S02 | canonical bytes/hash domain、create-exclusive version、event idempotency/conflict、full supersession chain 与 rollback 必须冻结到文件/接口级。 |
| CP5-FOCUS-CR163-003 | S03 | S03 必须同时覆盖 2 chains 与 CPI-001..004 4/4；wrapper/hook 同 chain 不得双建 session/trial。 |
| CP5-FOCUS-CR163-004 | S04 | 无 sealed native ref=typed_unavailable；manual mismatch=blocked；status 只可持平或变差；不得新建 gate 或填充 effective value。 |
| CP5-FOCUS-CR163-005 | S05 | 12/12 P0 scenarios、10 seals=1 hash、五类 negative 5/5 blocked、forbidden counters=0、CR155 blocked 1/1。 |

### First Slice / Deferred / Not Authorized

| 范围 | 状态 | 处理 |
|---|---|---|
| Six objects + session/event contract | first slice | S01；fixture/static 可验证。 |
| Append-only recorder/seal/supersession | first slice | S02；local repo artifacts only。 |
| Two producer chains / four mappings | first slice | S03 单 Story全覆盖。 |
| Existing consumer projection | first slice | S04；no new gate。 |
| Integrity/recovery/security/CR155 evidence | first slice | S05；static/fixture only。 |
| Effective trial / FDR/PBO/DSR/C1 computation | deferred/not authorized | 独立统计 CR。 |
| Historical CR155 backfill | not authorized | 只保留 blocked negative regression。 |
| Real data/runtime/credential/external registry | not authorized | 独立 authorization gate/CR。 |

### CR163 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 所有 Blueprint Feature 均有 required 或 waived-as-standalone 判定 | PASS | 本节 Feature 适用性判定 |
| required Feature 三件套完整 | PASS | `docs/features/experiment-family-lineage/`、`trial-lineage-producer-adapters/`、`strategy-admission-lineage-projection/` |
| 五 Story 均有 feature_design_refs 与 full-lld | PASS | 本节 Story 下游消费表 |
| FEAT-23 waiver 有理由、风险和重访条件 | PASS | standalone 不拥有接口；出现 validation service/schema 时升级 |
| CP4 不授权实现、测试实现、runtime/data/credential/statistical/backfill/external write | PASS | CR163 scoped development plan authorization boundary |

## CR164 CP4 增量：Computable Multiple-Testing / PBO / DSR Evidence

> 来源：CR164 CP3 已批准 HLD/ADR。只允许 Story、Feature 和 LLD 设计；不拉起子 Agent，不实现代码/测试，不执行真实统计、数据、runtime 或外部写入。

| Feature | 名称 | 判定 | 触发原因 | 三件套 | Stories | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|
| FEAT-24 | Statistical evidence contract | required | 公共 schema、sealed identity、availability、minima、canonical hash 被全部 Story 共享 | `docs/features/statistical-evidence-contract/{DESIGN,TEST-PLAN,TASKS}.md` | S01,S02,S03,S04,S05 | full-lld | schema/status/minima/hash 改变 |
| FEAT-25 | Multiple-testing calculators | required | BH 与 WRC/SPA 方法、fixed-window bootstrap、参数 provenance | `docs/features/multiple-testing-calculators/{DESIGN,TEST-PLAN,TASKS}.md` | S02,S05 | full-lld | automatic selector 或方法公式改变 |
| FEAT-26 | Overfit/deflation calculators | required | PBO/CSCV split contract、raw-count DSR 与 effective non-alias | `docs/features/overfit-deflation-calculators/{DESIGN,TEST-PLAN,TASKS}.md` | S03,S05 | full-lld | effective estimator 或 split policy 改变 |
| FEAT-27 | Conservative aggregation and projections | required | severity lattice、three consumers、no parallel gate | `docs/features/statistical-evidence-projection/{DESIGN,TEST-PLAN,TASKS}.md` | S04,S05 | full-lld | mandatory set、consumer schema 或 policy 改变 |
| FEAT-28 | Independent verification orchestration | waived-as-standalone | 不拥有生产接口或数据；S05 组合 FEAT-24..27 测试计划，独立设计会复制职责 | none；消费四套 TEST-PLAN | S05 | full-lld（Story） | 出现共享 validation API/service/runtime 时升级 required |

### Story 下游消费表

| Story | Owner Feature(s) | feature_design_refs | lld_policy | CP5 evidence |
|---|---|---|---|---|
| CR164-S01-statistical-evidence-contract-validator | FEAT-24 | `docs/features/statistical-evidence-contract/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | Story LLD |
| CR164-S02-bh-wrc-spa-evidence | FEAT-25/24 | FEAT-25 + FEAT-24 三件套 | full-lld | Story LLD |
| CR164-S03-pbo-cscv-dsr-evidence | FEAT-26/24 | FEAT-26 + FEAT-24 三件套 | full-lld | Story LLD |
| CR164-S04-conservative-aggregation-projections | FEAT-27/24 | FEAT-27 + FEAT-24 三件套 | full-lld | Story LLD |
| CR164-S05-independent-verification | FEAT-24..28 | 四套 TEST-PLAN/TASKS + FEAT-28 waiver | full-lld | Story LLD |

### CR164 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Blueprint Feature 判定覆盖 | PASS | FEAT-24..28 = 5/5 |
| required 三件套 | PASS | FEAT-24..27 = 4/4 |
| Story refs / lld_policy | PASS | 5/5 full-lld |
| verification waiver | PASS | 无新生产接口；出现 validation service 时重访 |
| 授权边界 | PASS | design-only、no-subagent、forbidden operations=0 |

## CR166 CP4 增量：Walk-forward / OOS Evidence Producer Foundation

> 来源：CR166 CP3 已批准 HLD/ADR。当前只允许 Story、Feature 与 LLD 设计；不启动子 Agent，不实现源代码/测试，不执行真实 fold/OOS、数据、runtime、外部操作或 Git remote write。

| Feature | 名称 | 判定 | 触发原因 | 三件套 | Stories | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|
| FEAT-166-01 | Strategy evidence envelope | required | 公共 canonical/envelope、C1 API/hash compatibility、C3/C4 extension catalog | `docs/features/strategy-evidence-envelope/{DESIGN,TEST-PLAN,TASKS}.md` | S01,S03,S05 | full-lld | schema/domain/hash/catalog 或 C1 public compatibility 变化 |
| FEAT-166-02 | Walk-forward/OOS validation | required | 时间与泄漏边界、7 字段族、daily/ML cross-contract、event N/A、authorization | `docs/features/walk-forward-oos-validation/{DESIGN,TEST-PLAN,TASKS}.md` | S02,S03,S05 | full-lld | calendar/session/event window、adapter input 或 leakage policy 变化 |
| FEAT-166-03 | Deterministic C2 producer | required | fold computation、declared denominator、hash/provenance/self-validation | `docs/features/walk-forward-oos-producer/{DESIGN,TEST-PLAN,TASKS}.md` | S03,S05 | full-lld | metric/outcome/aggregation/determinism contract 变化 |
| FEAT-166-04 | Existing-consumer projections | required | 三模块集成、backward compatibility、worse-state merge 与 CR155 claim ceiling | `docs/features/walk-forward-oos-projections/{DESIGN,TEST-PLAN,TASKS}.md` | S04,S05 | full-lld | consumer schema、mandatory policy、status lattice 或 package flags 变化 |
| FEAT-166-05 | Fixture/static verification | required | 跨四 Feature 的 12 QAC、安全/权限、CR155/CR165 回归归因属于 claim-sensitive 设计 | `docs/features/walk-forward-oos-verification/{DESIGN,TEST-PLAN,TASKS}.md` | S05 | full-lld | 新验证 runtime/service、真实数据 lane 或共享外部验证 API 出现 |

### CR166 Story 下游消费表

| Story | Owner Feature(s) | feature_design_refs | lld_policy | CP5 evidence |
|---|---|---|---|---|
| CR166-S01-evidence-envelope-contracts | FEAT-166-01 | FEAT-166-01 三件套 | full-lld | Story LLD |
| CR166-S02-fold-validator-adapters | FEAT-166-02/01 | FEAT-166-02 + FEAT-166-01 DESIGN/TEST-PLAN | full-lld | Story LLD |
| CR166-S03-deterministic-c2-producer | FEAT-166-03/02/01 | FEAT-166-03 + FEAT-166-02 + FEAT-166-01 DESIGN/TEST-PLAN | full-lld | Story LLD |
| CR166-S04-existing-consumer-projections | FEAT-166-04/03 | FEAT-166-04 + FEAT-166-03 DESIGN/TEST-PLAN | full-lld | Story LLD |
| CR166-S05-independent-static-verification | FEAT-166-05/01..04 | 五套 DESIGN/TEST-PLAN/TASKS | full-lld | Story LLD |

### CR166 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Blueprint Feature 判定覆盖 | PASS | FEAT-166-01..05 = 5/5 required |
| required Feature 三件套 | PASS | 5/5 × 3 文件完整 |
| Story refs / lld_policy | PASS | 5/5 full-lld；无 waived Story |
| 相邻边界 | PASS | C1 compatibility、C3/C4 calculator=0、event N/A、3 existing consumers |
| 授权边界 | PASS | design-only、no-subagent、external dereference/runtime/data/write=0 |

## CR168 CP4 增量：Economic Cost / Slippage / Impact Evidence Producer Foundation

> 来源：CR168 CP3 v1.1 HLD/ADR 已在吸收 A1–E3 深度评审后获用户批准。本阶段只允许 Feature/Story/LLD 设计，不拉起子 Agent，不修改 source/test，不执行真实数据、TCA、C4、runtime、aggregate、发布或 Git remote write。

### Feature 适用性判定

| Feature | 名称 | 判定 | 触发原因 | 三件套 | Stories | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|
| FEAT-168-01 | Economic Cost Evidence | required | 9-family validation、subject-neutral semantic hash、Decimal/rounding、static square-root、typed outcome 被 S01/S02/S03/S05 共享 | `docs/features/economic-cost-evidence/{DESIGN,TEST-PLAN,TASKS}.md` | S01,S02,S03,S05 | full-lld | schema/hash input domain、cost formula、rounding、availability 或 active family 变化 |
| FEAT-166-01 | Strategy Evidence Envelope（既有增量） | required-incremental | `economic_cost@reserved` 激活为 `economic_cost@v1`，且 attachment identity 必须由 envelope hash 独立绑定 | `docs/features/strategy-evidence-envelope/{DESIGN,TEST-PLAN,TASKS}.md` v0.2 | S01,S03,S05 | full-lld | neutral schema/domain/catalog、C1/C2 compatibility 或 attachment identity 变化 |
| FEAT-168-02 | Economic Cost Gate4 Projection | required | joint Gate4 C3/C4 payload、4-key allowlist、8-key denylist、candidate-release 和 postcondition guard 属于跨模块安全合同 | `docs/features/economic-cost-gate4-projection/{DESIGN,TEST-PLAN,TASKS}.md` | S04,S05 | full-lld | canonical public result、C4 refs、release profile、direct caller 或 aggregate path 变化 |
| FEAT-168-03 | Economic Cost Fixture Verification | required | 17 scenarios、15 QAC、10/10 negatives、2/2 fixtures、10→1 hash、CR155/no-runtime claim ceiling 是跨 Feature 的 claim-sensitive 验证设计 | `docs/features/economic-cost-verification/{DESIGN,TEST-PLAN,TASKS}.md` | S05 | full-lld | 新验证 runtime/service、真实 TCA/data lane、共享外部 API 或 independent verifier lane 出现 |

### Story 下游消费表

| Story | Owner Feature(s) | feature_design_refs | lld_policy | CP5 evidence |
|---|---|---|---|---|
| CR168-S01-c3-contract-identity-validation | FEAT-168-01 / FEAT-166-01 | Economic Cost Evidence 三件套 + Strategy Evidence Envelope DESIGN/TEST-PLAN | full-lld（data/security/cross-module） | Story LLD |
| CR168-S02-deterministic-cost-producer | FEAT-168-01 | Economic Cost Evidence 三件套 | full-lld（numeric/public contract） | Story LLD |
| CR168-S03-envelope-multi-strategy-compatibility | FEAT-166-01 / FEAT-168-01 | Strategy Evidence Envelope 三件套 + Economic Cost Evidence DESIGN/TEST-PLAN | full-lld（catalog/hash/backward compatibility） | Story LLD |
| CR168-S04-gate4-projection-containment | FEAT-168-02 / FEAT-168-01 | Gate4 Projection 三件套 + Economic Cost Evidence DESIGN | full-lld（security/cross-module/fail-closed） | Story LLD |
| CR168-S05-fixture-static-verification | FEAT-168-03 / FEAT-168-01 / FEAT-166-01 / FEAT-168-02 | 四 Feature 的 DESIGN/TEST-PLAN/TASKS | full-lld（cross-feature claim verification） | Story LLD |

### CP5 注意项

| Attention ID | Story | Requirement |
|---|---|---|
| CP5-FOCUS-CR168-001 | S01 | 九族全部校验；component hash 只绑定字段族 2-9 subject-neutral 成本语义；envelope hash 绑定 attachment identity；identity tamper false PASS=0。 |
| CP5-FOCUS-CR168-002 | S02 | proxy 合法域有限 `[0,+∞)` 且 `>1` 允许；五分项先 sum 再量化 total/net；minor unit 缺失 blocked；v1 只启用 static square_root。 |
| CP5-FOCUS-CR168-003 | S03 | catalog active schema=1；daily/ML 同成本语义 component hash 相同、不同 subject envelope hash 不同；C1/C2 regression=0。 |
| CP5-FOCUS-CR168-004 | S04 | candidate-release 固定；4-key allowlist、8-key presence denylist；escape canonical calls=0；PASS 与 postcondition violation 分 reason；canonical/aggregate changes=0。 |
| CP5-FOCUS-CR168-005 | S05 | requirements 9/9、scenarios 17/17、QAC 15/15、negative 10/10、fixtures 2/2、10 runs→1 hash、C4/capacity/aggregate PASS=0、CR155 promotion=0。 |

### First Slice / Deferred / Not Authorized

| 范围 | 状态 | 处理 |
|---|---|---|
| `economic_cost@v1` contract + static calculator | first slice | S01/S02；repo-local pure values only。 |
| Neutral envelope activation + daily/ML attach | first slice | S03；component semantics independent of package subject。 |
| CR168 adapter-local Gate4 containment | first slice | S04；one public caller、canonical/aggregate changes=0。 |
| Fixture/security/claim regression | first slice | S05；static-only。 |
| C4 producer/adapter present path | deferred | FU-CR161-005 decides adapter evolution。 |
| Canonical global hardening/C1-C4 aggregate/CR155 promotion decision | deferred | FU-CR161-007 or separate remediation CR。 |
| Real data/TCA/calibration/runtime/trading/publish/Git remote write | not authorized | 任何触达都需要独立授权。 |

### CR168 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Blueprint Feature 判定覆盖 | PASS | FEAT-168-01..03 + FEAT-166-01 = 4/4 required |
| required Feature 三件套 | PASS | 4/4 × 3 = 12/12 |
| Story refs / lld_policy | PASS | 5/5 full-lld；无 waived/technical-note |
| A1–E3 revision 下沉 | PASS | 五个 CP5 focus + Feature contract/test/tasks |
| 相邻边界 | PASS | C4=0、canonical/aggregate changes=0、event=0、CR155 promotion=0 |
| 授权边界 | PASS | design-only、no-subagent、source/test/external/runtime/write=0 |

## CR169 CP4 增量：Capacity / Liquidity / ADV Evidence Producer Foundation

> 来源：CR169 CP3 v1.1 HLD/ADR 已按用户评审补强后批准。当前只允许 Feature/Story/LLD 设计；CP5 批准前不修改 source/test、不执行新增测试，也不触达真实 ADV/liquidity、canonical Gate4、aggregate、Stage3 或远端 Git。

### Feature 适用性判定

| Feature | 名称 | 判定 | 触发原因 | 三件套 | Stories | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|
| FEAT-169-01 | Capacity / Liquidity / ADV Fixture Evidence | required | public typed component、Decimal static proxy、12-class fail-closed、hash/availability 被 S01/S02/S03/S05 共享 | `docs/features/capacity-liquidity-evidence/{DESIGN,TEST-PLAN,TASKS}.md` | S01,S02,S03,S05 | full-lld | schema/method/formula/rounding/hash/availability 变化 |
| FEAT-169-02 | C3/C4 Correlation Boundary | required | exact 13-field join、identity/envelope 分域、fail-before-canonical 属于跨组件安全合同 | `docs/features/c3-c4-correlation-boundary/{DESIGN,TEST-PLAN,TASKS}.md` | S01,S04,S05 | full-lld | header 字段、view 来源、temporal/audit 规则变化 |
| FEAT-169-03 | Strict C3+C4 Gate4 Fixture Adapter | required | 7-key allowlist、reason escape、public callable DI、postcondition 与 forbidden-write 边界 | `docs/features/capacity-liquidity-gate4-joint-adapter/{DESIGN,TEST-PLAN,TASKS}.md` | S04,S05 | full-lld | canonical public result、Gate4 fields、release profile 或 aggregate caller 变化 |
| FEAT-169-04 | C4 Fixture Claim / Stage2 Exit Guard | required | 17 scenarios、15 QAC、12/12 P0、CR155/CR168 回归、7/7 exit 证据是 claim-sensitive 跨 Feature 验证 | `docs/features/capacity-liquidity-claim-exit-guard/{DESIGN,TEST-PLAN,TASKS}.md` | S05 | full-lld | Stage2 exit contract、真实数据 lane、independent verifier 或 release claim 变化 |

### Story 下游消费表

| Story | Owner Feature(s) | feature_design_refs | lld_policy | CP5 evidence |
|---|---|---|---|---|
| CR169-S01-capacity-liquidity-contract-validation | FEAT-169-01/02 | Evidence + Correlation 三件套 | full-lld（public contract/hash/security） | Story LLD |
| CR169-S02-deterministic-static-c4-producer | FEAT-169-01 | Evidence 三件套 | full-lld（numeric/public producer） | Story LLD |
| CR169-S03-neutral-envelope-c4-activation | FEAT-169-01 / CR166 envelope | Evidence 三件套 + existing envelope DESIGN/TEST-PLAN | full-lld（catalog/hash/backward compatibility） | Story LLD |
| CR169-S04-strict-c3-c4-gate4-joint-adapter | FEAT-169-02/03 | Correlation + Joint Adapter 三件套 | full-lld（security/cross-module/postcondition） | Story LLD |
| CR169-S05-fixture-claim-stage2-exit-verification | FEAT-169-04/01/02/03 | 四 Feature 的 DESIGN/TEST-PLAN/TASKS | full-lld（cross-feature claim verification） | Story LLD |

### CP5 注意项

| Attention ID | Story | Requirement |
|---|---|---|
| CP5-FOCUS-CR169-001 | S01 | N01..N12 精确 reason、13 字段 header、identity 只入 envelope、issue path calculator=0。 |
| CP5-FOCUS-CR169-002 | S02 | `static_adv_cap_v1` 三条公式、Decimal precision=28、minor-unit HALF_EVEN、cap `(0,1]`、3 refs typed present。 |
| CP5-FOCUS-CR169-003 | S03 | catalog 仅激活 `capacity_liquidity@v1`；C1/C2/C3 descriptors/hash 不变；daily/ML component/envelope identity 分域。 |
| CP5-FOCUS-CR169-004 | S04 | 13-field precheck、exact 7-key、reason/extra-key reject、public Protocol DI、candidate-release、non-upgrading postcondition。 |
| CP5-FOCUS-CR169-005 | S05 | 9/9 REQ、17/17 scenarios、15/15 QAC、12/12 P0、2/2 fixtures、10→1、CR155 blocked、Stage2 exit 7-item result shape。 |

### First Slice / Deferred / Not Authorized

| 范围 | 状态 | 处理 |
|---|---|---|
| C4 typed contract + static proxy | first slice | S01/S02；pure repository-local values only。 |
| Neutral envelope activation | first slice | S03；不修改既有 C1/C2/C3 semantic contracts。 |
| Strict C3+C4 fixture adapter | first slice | S04；canonical read-only，aggregate=0。 |
| Fixture/claim/exit verification | first slice | S05；Stage2 result 在 CP8 生成。 |
| Alpha decay | deferred | FU-CR161-008；calculator=0。 |
| Canonical global N/A hardening / aggregate / CR155 promotion | deferred | FU007a/b future tracking；不启动。 |
| Real data/runtime/trading/publish/Git remote write | not authorized | 必须独立授权。 |

### CR169 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Blueprint Feature 判定覆盖 | PASS | FEAT-169-01..04 = 4/4 required |
| required Feature 三件套 | PASS | 4/4 × 3 = 12/12 |
| Story refs / lld_policy | PASS | 5/5 full-lld；无 waived/technical-note |
| DAG/Wave | PASS | 5 nodes、10 edges、5 serial Waves、cycles=0、parallel conflicts=0 |
| CP3 评审补强下沉 | PASS | exact header、callable、Stage2 failure route、FU008、method basis |
| 授权边界 | PASS | design-only；source/test/external/runtime/remote write=0 |

## CR170 CP4 增量：Canonical Reliability N/A Semantics and Admission Hardening

> 来源：CR170 CP3 approved HLD/ADR。当前只允许 Feature/Story/LLD 设计；CP5 批准前不修改 source/test、不执行新增测试，不触达真实数据、Stage3 runner、aggregate、CR155 promotion、runtime/trading 或 Git remote write。

### Feature 适用性判定

| Feature | 名称 | 判定 | 触发原因 | 三件套 | Stories | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|
| FEAT-15 | Cross-Strategy Production Reliability Gates | required-increment | 修改 canonical mandatory evidence 语义、Gate1-5 consumer、Gate6 admission policy；涉及公共兼容、安全 fail-closed、跨 Story contract | `docs/features/cross-strategy-reliability-gates/{DESIGN,TEST-PLAN,TASKS}.md` v0.2 | S01,S02,S03,S04 | full-lld | policy unit、五态、public callable/schema、tier、adapter 或 real caller 边界变化 |

不新增平行 Feature：CR-170 修改的是 FEAT-15 的既有 canonical owner。future verifier 属 FU-006，aggregate/adapter simplification 属 FU-007b/FU-009，均不进入本轮。

### Story 下游消费表

| Story | Owner Feature | feature_design_refs | lld_policy | CP5 evidence |
|---|---|---|---|---|
| CR170-S01-na-policy-inventory-five-state-contract | FEAT-15 policy contract | FEAT-15 三件套 | full-lld（安全语义、21-unit contract） | Story LLD |
| CR170-S02-gate1-5-na-consumer-hardening | FEAT-15 Gate consumers | FEAT-15 三件套 | full-lld（canonical shared module、双向爆炸半径） | Story LLD |
| CR170-S03-admission-worst-state-tier-hardening | FEAT-15 Gate6/admission | FEAT-15 三件套 | full-lld（共享文件、tier、安全兼容） | Story LLD |
| CR170-S04-compatibility-claim-closure | FEAT-15 compatibility | FEAT-15 DESIGN/TEST-PLAN/TASKS | full-lld（跨组件 claim 回归） | Story LLD |

### CP5 强制注意项

| Attention ID | Story | Requirement |
|---|---|---|
| CP5-FOCUS-CR170-001 | S01 | 21/21 exact mapping；Gate 6/6/1/5/3；baseline path type + direction + disposition；15/5/1；five-state 5/5。 |
| CP5-FOCUS-CR170-002 | S01 | `n_a_boundaries` 当前 writer 仅 fixture/test；evaluator synthesis=0；`authorization_ref` 仅 opaque audit pointer。 |
| CP5-FOCUS-CR170-003 | S02 | stricter/controlled-widening/preserve 三组分向回归；Gate1 classifier/claim/final status 3/3。 |
| CP5-FOCUS-CR170-004 | S03 | protected merge 先回归，成功则 production diff=0；T0/T1/T2=NR/BLOCKED/BLOCKED；T3 early-return 1/1 且 diff=0。 |
| CP5-FOCUS-CR170-005 | S04 | public break=0、adapter 2/2、guard deletion=0、CR155 promotion=0、runner/aggregate/real-op=0。 |

### DAG / Wave / File Owner

| Wave | Story | depends_on | 主要 owner | 并行结论 |
|---|---|---|---|---|
| CR170-W1-POLICY | S01 | none | `engine/reliability_na_policy.py` | serial |
| CR170-W2-GATE-CONSUMERS | S02 | S01 contract | `engine/cross_strategy_reliability_gates.py` | serial |
| CR170-W3-ADMISSION | S03 | S01+S02 contract/runtime/file-conflict | 同一 canonical 文件，S03 为第二写入方 | serial |
| CR170-W4-COMPATIBILITY | S04 | S02+S03 runtime | CR170 regression tests | serial |

DAG：4 nodes、5 edges、4 serial Waves、cycles=0、invalid refs=0、parallel file conflicts=0。由于用户禁止子 Agent，`max_parallel_lld/dev/qa=1`。

### First Slice / Deferred / Not Authorized

| 范围 | 状态 | 处理 |
|---|---|---|
| 21-unit policy + Gate1-5 consumers + admission hardening | first slice | S01-S03；repository-local、无 IO。 |
| compatibility/claim closure | first slice | S04；read-only regression，adapter production diff=0。 |
| independent verifier | deferred | FU-006 future consumer；当前由 Gate maintainer 自验证，CP8 披露。 |
| aggregate / adapter simplification / Stage3 launch | deferred | FU-007b/FU-009/独立 Stage3 CR。 |
| real data/runtime/trading/publish/Git remote write | not authorized | 独立人工授权。 |

### CR170 CP4 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| Feature owner/三件套 | PASS | FEAT-15 required-increment；v0.2 DESIGN/TEST-PLAN/TASKS = 3/3 |
| Story refs / lld_policy | PASS | 4/4 full-lld；无 waived/technical-note |
| Scenario coverage | PASS | 20/20 分配到 S01-S04；REQ 9/9、QAC 15/15 |
| DAG/Wave | PASS | 4 nodes、5 edges、4 serial Waves、cycles=0、parallel conflicts=0 |
| 评审整改 | PASS | 21-unit path/direction、caller contract、T3 zero-diff 均下沉到 CP5 focus |
| 授权边界 | PASS | design-only；source/test/real-data/runtime/remote-write=0 |

## CR173 CP4 增量：Effective-Trial Offline Estimator

> 来源：CR173 CP3 用户批准的 `spectral_participation_ratio` 限定 claim 与 estimator-only split。当前只准备 Feature/Story/CP5 设计证据队列；不创建 LLD，不修改源码/测试/fixture，不发起 CP5，不触达 public C1、真实数据或 runtime。

### Feature 适用性判定

| Feature | 名称 | 判定 | 触发原因 | 三件套 | Stories | lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|
| FEAT-173-01 | Effective-Trial Offline Estimator | required | 新 typed input/evidence schema、exact-rational PSD/numeric algorithm、versioned serialization、fail-closed/append-only contract 被 3 个 Story 共享 | `docs/features/effective-trial-offline-estimator/{DESIGN,TEST-PLAN,TASKS}.md` | S01,S02,S03 | full-lld | estimand、matrix representation、numeric/PSD、七字段、reason/hash、fixture schema、authorization 或 public boundary 变化 |

只创建一个 estimator-only Feature。public C1 projection 已按 CP3 `PASS_BY_SPLIT` 留在后续 CR 候选，不是 FEAT-173-01 的 capability、Story、task 或实现文件；production projection Story 数=`0`。

### Story 下游消费表

| Story | Owner Feature | feature_design_refs | lld_policy | CP5 evidence | 说明 |
|---|---|---|---|---|---|
| CR173-S01-contract-evidence-canonicalization | FEAT-173-01 contract/evidence | FEAT-173-01 三件套 | full-lld（schema/serialization/security/shared contract） | Story LLD | 冻结 immutable input/method/7-field evidence、8 failure reason、canonical bytes/hash；public write=0。 |
| CR173-S02-exact-spectral-estimator | FEAT-173-01 estimator | FEAT-173-01 三件套 | full-lld（exact numeric/PSD/public method semantics） | Story LLD | 只消费 S01 contract；exact rational LDLT、`n²/ΣRij²`、一次舍入与双 invariant。 |
| CR173-S03-golden-failure-boundary-verification | FEAT-173-01 verification | FEAT-173-01 三件套 | full-lld（cross-story validation/authorization/claim ceiling） | Story LLD | 6×3、8 failures、append-only recovery、8+12 public inventory、zero-operation guard；无 projection。 |

本轮只登记 `lld_policy` 与未来 evidence path；实际 LLD 文件数=`0`，CP5 人工门禁发起数=`0`。

### CP5 强制注意项

| Attention ID | Story | Requirement |
|---|---|---|
| CP5-FOCUS-CR173-001 | S01 | 七字段 `7/7`、present/null/ref规则、8 failure reason、append-only recovery 与 canonical domain 必须逐项可执行；`Decimal→float=0`。 |
| CP5-FOCUS-CR173-002 | S02 | pivot comparator/zero-pivot residual coupling 必须有确定伪代码；exact rational/tolerance=0；未舍入+舍入后范围检查 `2/2`；禁止 clamp。 |
| CP5-FOCUS-CR173-003 | S03 | golden `6/6×3/3`、failure `8/8`、public inventory `8+12`、forbidden counters 全为 0；static guard 不得创建 public adapter。 |
| CP5-FOCUS-CR173-004 | ALL | public C1/Gate1/statistical summary/DSR/admission production diff/call=`0/0`；strategy/real/provider/credential/runtime/write/trading/remote=`0`。 |
| CP5-FOCUS-CR173-005 | ALL | `offline_method_ready` 不得提升为 public computable、Stage3、admission 或 CR172 auto-resume；public projection Story/Feature/task=`0/0/0`。 |

### DAG / Wave / File Owner

| Wave | Story | depends_on | dependency type | 主要 owner | 并行结论 |
|---|---|---|---|---|---|
| CR173-W1-CONTRACT-EVIDENCE | S01 | none | none | `engine/effective_trial_evidence.py` + contract tests | serial |
| CR173-W2-EXACT-ESTIMATOR | S02 | S01 | contract | `engine/effective_trial_estimator.py` + estimator tests | serial after S01 contract |
| CR173-W3-GOLDEN-BOUNDARY | S03 | S01,S02 | runtime,runtime | CR173 fixture/QAC/authorization tests | serial after S01/S02 evidence |

DAG：3 nodes、3 edges、3 serial Waves、cycles=0、invalid refs=0、isolated nodes=0、parallel internal dependencies=0、same-wave file conflicts=0。merge order=`S01 → S02 → S03`。

### REQ / SC / Outcome 覆盖

| Story | REQ-CR173 | SC-CR173 | Product outcomes |
|---|---|---|---|
| S01 | 001,002,003,004,005,007 | Q01,F01,N01,B01,C01 | O01,O02,O04 |
| S02 | 001,002,003,005,006 | P01,Q01,F01,B01,D01 | O01,O03 |
| S03 | 003,004,005,006,007,008 | F01,N01,B01,D01,C01,A01 | O02,O03,O04 |

并集：REQ `8/8=100%`，SC `8/8=100%`，Outcome `4/4=100%`。SC-C01 是 public-boundary stop 的 negative/static guard，不是 projection Story。

### First Slice / Deferred / Not Authorized

| 范围 | 状态 | 处理 |
|---|---|---|
| contract/evidence + exact estimator + fixture verification | first slice | S01→S02→S03；仅 CP5 后 repository-local fixture 实现。 |
| empirical matrix/stability/alpha-tail calibration | deferred | future methodology Spike/activation evidence；CR173 实现数=0。 |
| public C1 versioned projection/migration | deferred-split | future独立 CR；当前 Feature/Story/task/write/call均为0。 |
| real data/strategy/provider/credential/runtime/trading/publish/remote write | not authorized | 不建 Story、不建 task、不执行。 |

### CR173 CP4 准备自检与建议

| 检查项 | 结果 | 证据 |
|---|---|---|
| Feature owner/三件套 | PASS | FEAT-173-01 required；DESIGN/TEST-PLAN/TASKS=`3/3` |
| Feature/Story/Wave/Task 一致 | PASS | `1/3/3/12`，三件套、Story cards、DEVELOPMENT-PLAN一致 |
| Story refs / lld_policy / acceptance / failure route | PASS | `3/3` full-lld；必填消费字段与门控齐备；LLD实际文件=0 |
| REQ/SC/Outcome coverage | PASS | `8/8`、`8/8`、`4/4`，未覆盖=0 |
| DAG/Wave/file owner | PASS | 3 nodes、3 edges、3 Waves、cycle/ref/conflict=`0/0/0` |
| public projection split | PASS | projection Feature/Story/task/production write/call=`0/0/0/0/0` |
| 授权边界 | PASS | design-only；source/test/fixture/real/runtime/public/remote write=`0` |

meta-se 对 Host 的 CP4 建议：`PASS`，blocker=`0`。正式 CP4 result/context/checkpoint ledger 由 Host Orchestrator 生成；本增量不发起 CP5 门禁。

## CR172 PATH-I CP4 增量：Trial-Return 与跨机部署合同

> 来源：CR172 PATH-I HLD/ADR v1.3（CP5 R2 minimal correction）。本增量只准备 Feature/Story/CP5 设计证据队列和 repository-local fixture 计划；不修改现有 runner/lineage，不执行六类真实动作，不恢复 PATH-C/A。

### Feature 适用性判定

| Feature | 名称 | 判定 | 触发原因 | 三件套 | Stories | lld_policy |
|---|---|---|---|---|---|---|
| FEAT-CR172-I01 | Trial Return Artifact Pipeline | required | 新 immutable schema、fixture producer port、manifest/seal/verifier、partial-lineage BLOCKED audit | `docs/features/trial-return-artifact-pipeline/{DESIGN,TEST-PLAN,TASKS}.md` | S02,S05 | full-lld |
| FEAT-CR172-I02 | Research Artifact Replica and Materialization | required | 跨机副本、receipt、原子 pointer、执行机 immutable cache 与恢复 | `docs/features/research-artifact-replica-materialization/{DESIGN,TEST-PLAN,TASKS}.md` | S03,S04,S05 | full-lld |
| FEAT-CR172-I03 | PATH-I Authorization and Claim Governance | required | 六动作分权/执行资格 DAG、empirical/path/signal/claim shared contract | `docs/features/path-i-authorization-claim-governance/{DESIGN,TEST-PLAN,TASKS}.md` | S01,S05 | full-lld |

判定结果：required/waived/n/a=`3/0/0`；三件套=`9/9`。Signal detailed exchange/intraday、FU-CR173-001 v2、external import activation、真实 migration/runtime 的 Feature/Story/task=`0/0/0`。

### Story 下游消费表

| Story | Owner Feature | feature_design_refs | lld_policy | CP5 evidence | 主要文件 owner |
|---|---|---|---|---|---|
| CR172-S01-action-authorization-eligibility-governance | I03 | I03 三件套 | full-lld | Story LLD | `engine/path_i_governance.py` |
| CR172-S02-trial-return-artifact-native-hook | I01+I03 | I01/I03 六件 | full-lld R2 | Story LLD | `engine/trial_return_artifact.py`；历史 ID 保留，runner diff=0 |
| CR172-S03-nas-replica-verification | I02+I03 | I02/I03 六件 | full-lld | Story LLD | `engine/research_artifact_replica.py` |
| CR172-S04-execution-cache-materialization | I01+I02+I03 | I01/I02/I03 九件 | full-lld R3 | Story LLD | `engine/research_artifact_materialization.py` |
| CR172-S05-path-i-integration-claim-zero-operation-verification | I01+I02+I03 | 三 Feature 九件 | full-lld | Story LLD | CR172 test/fixture paths only |

实际 LLD 文件数=`0`；CP5 人工门禁发起数=`0`。五个 Story 全部 `lld-ready`，但实现/测试在 CP5 批准前均 blocked。

### `DO-CR172-CP5-001` 安全收窄闭环

| 义务 | CP4 冻结值 | 证据 |
|---|---|---|
| current source disposition | native multi-trial period-return producer=`0`；existing runner hook/diff=`0/0` | I01 DESIGN §2/§3；ADR-011 |
| semantic guard | `forward_label_proxy@v1`→trial-return/empirical-R/effective-count accepted=`0/0/0` | I01 DESIGN §2/§6 |
| source/file-owner inventory | `100%/100%`；planned touched code/test `2/2`，duplicate primary owner=0 | I01 DESIGN §3；S02 |
| fixture binding | decision-origin/target-kind=`2/2`；fixture+real accepted=0 | I01 DESIGN §5；I03 DESIGN §3 |
| seal handoff | canonical bytes/digest/verifier truth=`1/1/1`；S03 secondary digest=0 | I01 DESIGN §4；I02 DESIGN §2/§3 |
| lineage failure | partial success→`partial_lineage_blocked_audit`；canonical selection advance=0 | I01 DESIGN §6；ADR-009/011 |

### CP5 R3 四项机械闭环

| 项 | 冻结值 |
|---|---|
| approved-ledger | current-v1 authorized/eligible=`false/false`；reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；caller self-assert unlock=0 |
| S04 seal verification | data source=S03 selected replica；verifier=S02唯一 library/窄 facade；bypass/secondary digest=`0/0` |
| fixture provenance | 唯一来源=`ActionDecisionV1.decision_origin + ActionScopeContextV1.target_kind + fixture URI/port`；`evidence_kind` truth=`0` |
| REQ-013 | `contract_ready/runtime_enforcement_deferred`；current default switch/enforcement=`0/0`；future native-producer path-enforcement required |

真实 producer/instrumentation 与原子 lineage/correction 均为独立前置 CR；不得从历史 Story ID 恢复 runner hook。

### DAG / Wave / File Owner

| Wave | Story | depends_on | dependency type | 主要 owner |
|---|---|---|---|---|
| CR172-W1-GOVERNANCE-CONTRACT | S01 | none | none | governance contract/tests |
| CR172-W2-TRIAL-RETURN-HOOK | S02 | S01 | contract | artifact module + fixture port；历史 Wave ID 保留，runner diff=0 |
| CR172-W3-NAS-REPLICA | S03 | S01,S02 | contract,runtime | replica module/tests |
| CR172-W4-EXECUTION-MATERIALIZATION | S04 | S01,S02,S03 | contract,verifier-library,runtime-selection | materialization module/tests |
| CR172-W5-INTEGRATED-VERIFICATION | S05 | S01,S02,S03,S04 | runtime×4 | test/fixture only |

DAG=`5 nodes/10 edges/5 serial Waves`；cycles/invalid refs/isolated/same-wave conflicts=`0/0/0/0`；merge order=`S01→S02→S03→S04→S05`。

### REQ / Scenario / Outcome 覆盖

| Story | Requirement | Scenario | Outcome |
|---|---|---|---|
| S01 | 001～008,011,013～015 | P01,P02,N01,B01,F01,Q01,A01,G01,A02,Q02,G02,C02,S01～S06 | O01～O04,O07,O09～O11 |
| S02 | 009,011,013,014 | I01,N02,N04,B02,G02,Q02 | O05,O09,O11 |
| S03 | 010～012 | I02,N03,B02,B03,A02,F02 | O06～O08,O11 |
| S04 | 010～012 | I02,N03,B02,B03,A02,F02 | O06～O08,O11 |
| S05 | 001～015 | all CR172 P0 scenarios | O01～O11 |

并集：REQ=`15/15=100%`、Scenario=`27/27=100%`、Outcome=`11/11=100%`；uncovered=`0/0/0`。

### CP5 强制注意项

1. S01 必须分别表达 `authorized` 与 `eligible_to_execute`；DAG edges `5/5`，runtime-without-read launch/workspace/pointer=`0/0/0`。
2. S02 必须保持 current runner/lineage diff=`0/0`、v1 exact 两列、forward-label accepted=0、fixture binding、verified seal public contract 与 partial-lineage BLOCKED audit。
3. S03 必须精确消费 S02 verified result；S04 只能从 S03 selection 取数，但必须复用唯一 verifier 做 bytes-level 复验；bypass/re-seal/secondary digest/direct-NAS/pointer escape=0。
4. S05 provenance 只消费 decision origin + target kind + fixture URI/port，`evidence_kind`=0；approved-ledger 双 false；REQ-013 runtime delivered claim=0。
5. Signal detailed/FU-v2/external import/real runtime/migration Story 数持续为 0；发现需要即回独立 CR/设计澄清。

### CR172 CP4 自检与建议

| 检查项 | 结果 | 证据 |
|---|---|---|
| Feature/三件套 | PASS | `3/9`（3 Feature、9 files） |
| Story/Wave/Task | PASS | `5/5/20`；全部 full-lld；LLD actual=0 |
| coverage | PASS | REQ/SC/Outcome=`15/27/11` 全覆盖 |
| DAG/file owner | PASS | 5 nodes、9 edges；cycle/ref/conflict/duplicate owner=0 |
| source disposition | PASS | current runner hook/diff=`0/0`；forward proxy accepted=0；future producer/atomic lineage prerequisites registered |
| R3 minimal guards | PASS | approved-ledger accepted/eligible=0/0；S04 verifier chain=1；evidence_kind=0；REQ-013 runtime enforcement=deferred |
| authorization/scope | PASS | authorized/executed=`0/6`,`0/6`；deferred scope Story=0 |

meta-se 对 Host 的 CP4 建议：`PASS`，blocker/waiver/新人工决策=`0/0/0`；下一步由 Host 在 CP4 PASS 后调度 5 份 full LLD 并发/串行策略，不在本增量发起 CP5。
