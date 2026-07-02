---
title: "Strategy Framework Implementation Assessment and Remediation Plan"
status: "draft-v0.3"
owner: "host-orchestrator"
created_at: "2026-07-01"
scope: "multifactor, machine-learning, event-driven strategy E2E framework remediation"
reviews: "docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md"
roadmap_ref: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md"
authorization_boundary: "planning-only; no source implementation, runtime, lake write, NAS sync, provider fetch, QMT, broker, credential, simulation, live, or trading authorization"
---

# Strategy Framework Implementation Assessment and Remediation Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-07-01 | host-orchestrator | 初版：基于 E2E framework review、CR150 completion map 与用户决策 A，评估当前多因子实现，并给出多因子、ML、事件驱动和横切统计可靠性整改计划。 |
| v0.2 | 2026-07-01 | host-orchestrator | 补 CR151 wave 切分建议、CP0/CP2 启动约束和 ML/event 方法论来源声明，避免 Future CR 范围过大或被误读为已立项。 |
| v0.3 | 2026-07-02 | host-orchestrator | 回写 CR151 CP8 `READY_WITH_RISK` 闭环与 opt-in gate 性质；刷新 CR152 CP2 必决项：ML gate 与 CR151 gate 关系、statistical gate opt-in/强制时机；新增 STATE/CR-INDEX/tool-check hygiene 并行治理建议。 |

## 1. 当前决策

用户选择路线 A：

> 先补齐本地研究 / 策略框架；数据湖生产级闭环、NAS strict metadata parity、business-conflict quarantine 解除和 provider/runtime/broker 相关工作后置。

本计划因此只覆盖设计、契约、fixture、静态验证和本地框架补齐路线，不授权任何真实数据湖、NAS、provider、QMT、simulation/live/trading、broker、credential 或外部框架运行。

## 2. 事实基线

### 2.1 已完成事实

| 范围 | 当前事实 |
|---|---|
| CR147 research production foundation | 已有 `ResearchDatasetSpec`、leakage policy、snapshot / feature availability contract 等源码级契约入口。 |
| CR148 backtest foundation | 已有 `BacktestRunSpec`、report pack、cost/risk/attribution metadata contract 等源码级契约入口。 |
| CR150 multifactor completion | 已有 `MultifactorFrameworkCompletionMap`，将 factor spec、factor run、factor panel、label window gate、signal set、portfolio policy、BacktestRunSpec、report pack、cost/risk attribution、strategy admission package 连成 metadata linkage chain。 |
| CR150 validation boundary | CP8 已以 `READY_WITH_RISK` 收尾；有效验证模式是 `static-only`，不代表真实 lake、runtime、simulation、live 或 trading 证据。 |
| Data lake local baseline | CR146 / CR149 已让本地 research current truth、PIT/readiness、business-conflict quarantine policy 具备可治理状态。 |
| NAS content sync | RA-CR149-001 已完成 scoped sync 内容层对齐；strict permissions / group metadata parity 后置为 FU-CR149-002。 |
| CR151 statistical admission gate | 已关闭为 `READY_WITH_RISK`：`StrategyAdmissionStatisticalGate`、multiple-testing、robust statistics、walk-forward/OOS、PBO/DSR、admission package linkage 和 optional completion-map linkage 已完成；`CR151-CP8-R03-STATE-V2-HYGIENE` 已由用户接受为 process caveat。 |

### 2.2 E2E Review 需要校正的事实

`docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` 中关于 `ResearchDatasetSpec` / `BacktestRunSpec` “.py 文件不存在”的判断已不符合当前源码状态：

| 契约 | 当前源码 |
|---|---|
| `ResearchDatasetSpec` | `engine/research_production_contracts.py` |
| `BacktestRunSpec` | `engine/backtest_production_contracts.py` |

后续整改不应重复创建平行契约，而应复用并扩展现有源码契约。E2E review 中关于统计可靠性、walk-forward / OOS、多重检验、PBO/DSR、市场冲击、容量、ML 专项方法和事件研究方法的 blocker 仍然成立。

## 3. 多因子实现评估

### 3.1 已实现能力

| 能力 | 当前实现情况 |
|---|---|
| Asset inventory / linkage map | `MultifactorFrameworkCompletionMap` 的 nodes 提供 asset inventory，`linkage_gaps` 提供 gap-count 等价视图。 |
| Metadata chain | factor spec 到 strategy admission package 的 10 节点链路已经可构建、可 hash、可验证。 |
| Boundary control | CR150 forbidden operations 为零；deferred routes 显式列出 lake/NAS/provider/QMT/runtime/broker/credential/external framework 等未授权面。 |
| Static validation | py_compile、pytest、JSON、workspace、cr-tracking、git diff 等静态验证已通过。 |
| Stage3 mature runner baseline | 现有 mature multifactor runner 已覆盖 IC / RankIC、layered returns、turnover、exposure、risk policy、capacity proxy、strategy admission package 等基础研究工件。 |
| Source contract availability | ResearchDatasetSpec 与 BacktestRunSpec 已经落到源码，不是只存在于 evidence JSON。 |

### 3.2 仍未达到生产级的缺口

| 缺口 | 影响 | 建议处理 |
|---|---|---|
| 多重检验 / 数据窥探门缺失 | 多个因子或参数试验容易产生伪阳性，IC / Sharpe 不能直接作为准入依据。 | 在 CR151 增加 FDR/BH、White/Hansen 或等价多重检验工件，并进入 admission gate。 |
| Newey-West 或等价稳健统计缺失 | IC / return 序列存在自相关时，普通 t 统计显著性偏乐观。 | 增加自相关稳健 t-stat / p-value 报告。 |
| Walk-forward / OOS 非一等阶段 | 单全样本回测无法区分预测力与 curve-fit。 | 增加 rolling / walk-forward plan、OOS fold manifest、fold-level metrics。 |
| PBO / DSR 缺失 | 回测过拟合与选择偏差没有量化阻断。 | 增加 PBO/CSCV 或近似 fixture 实现，以及 Deflated Sharpe Ratio 输入工件。 |
| Universe PIT 审计不足 | 未来退市、ST、指数成分变化会造成 survivorship bias。 | 增加 PIT universe audit refs 与退市 / ST 纳入日志。 |
| 因子相关性聚类不足 | 冗余因子会重复押注同一风险源。 | 增加 factor correlation clustering / redundancy report。 |
| Market impact / capacity 仍偏 proxy | commission/tax/slippage 之外的冲击成本与容量上限不足。 | 增加 ADV participation、capacity dollars、square-root / Almgren-Chriss style model spec。 |
| 相对基准指标不足 | 绝对收益无法区分 beta 与 alpha。 | 增加 IR、tracking error、Active Share 或等价 benchmark-relative report。 |

## 4. 推荐下一 CR：CR151 Strategy Research Admission Statistical Gate

### 4.1 目标

把 CR150 的 metadata linkage completion 升级为可审计的本地 strategy research admission framework，使多因子研究输出不只“链路完整”，还具备最低统计可靠性门。

### 4.2 范围

In scope：

1. 复用现有 `ResearchDatasetSpec` / `BacktestRunSpec` / `StrategyAdmissionPackage`。
2. 新增或扩展统计验证契约与报告对象。
3. 用 fixture / static-only / deterministic local examples 验证门控语义。
4. 更新 roadmap、evidence、CP result 和 release readiness。

Out of scope：

1. 真实 lake 读取或写入。
2. NAS sync / chmod / chgrp / metadata normalization。
3. provider fetch。
4. QMT / MiniQMT / xtquant runtime。
5. simulation、paper、live、trading、broker write。
6. credential read。
7. 外部框架真实运行。

### 4.3 Wave 切分建议

CR151 当前是计划层候选，CP2 前应先做 wave 切分判断，避免单个 CR 同时承载过多统计、归因、容量和数据治理工件。推荐切分如下：

| Wave | 范围 | 理由 |
|---|---|---|
| CR151 Wave A：admission gate 硬前置 | FDR/BH 或等价多重检验、Newey-West 或等价稳健 t-stat、walk-forward/OOS fold manifest、PBO/DSR、`StrategyAdmissionStatisticalGate` | 这些直接决定一个因子 / 策略是否可被准入，是 gate 本体。 |
| CR151 Wave B 或 CR151.5：评价与横切工件 | factor correlation clustering、PIT universe audit、capacity/impact report、IR/TE/Active Share、benchmark-relative report | 这些对生产质量重要，但可在 admission gate 核心闭环后作为评价、容量或横切生产可靠性工件补齐。 |

若 CP2/CP3 发现 Wave B 与 CR154 横切生产可靠性重叠，应优先把 capacity/impact、benchmark-relative 和 regime/attribution 类工件移入 CR154，CR151 保持“准入统计门”小切片。

### 4.4 当前状态与 opt-in 性质

CR151 已于 2026-07-02 通过 CP8，并以 `READY_WITH_RISK` 关闭。

当前 capability 判断：

| 能力 | 状态 | 说明 |
|---|---|---|
| 多重检验 / FDR | complete | `MultipleTestingReport` 与 validation helper 已实现并测试。 |
| Newey-West / robust stats 最小门 | complete | `RobustFactorStatisticsReport` 覆盖 Wave A 最小稳健统计入口。 |
| Walk-forward / OOS | complete | `WalkForwardValidationPlan` 与 fold-level gate 已实现。 |
| PBO / DSR | complete | `BacktestOverfitRiskReport` 已进入 aggregate evaluator。 |
| Fail-closed aggregate gate | complete | `StrategyAdmissionStatisticalGate` 四态语义和 forbidden counter precedence 已测试。 |
| Admission package linkage | complete | `attach_statistical_gate_to_admission_package` 和 status mapping 已实现。 |
| Completion-map linkage | complete / opt-in | `build_cr150_multifactor_framework_completion_map(..., statistical_gate_summary=None, require_statistical_gate=False)` 默认保持 CR150 历史行为不变。 |

重要边界：CR151 完成的是 statistical admission capability，不是把 UC-58 默认流程改成强制统计门。`require_statistical_gate=False` 是有意默认值，用于避免破坏 CR150 历史 completion-map 行为。若要把 UC-58 / UC-59 调用方改为必须经过 statistical gate，应在 CR152 CP2 或单独 admission governance 中显式决策。

### 4.5 启动约束（历史记录）

CR151 尚未作为 formal CR 立项。真正启动前必须先经 CP0 受理和 CP2 场景 / 范围基线门；CP2 未通过前不得进入 Story 拆解、LLD 或实现设计。若 CR151 继续保持 static/fixture 验证模式，应在 CP2 Decision Brief 中预先说明验证模式、meta-qa/subagent 验证方式，以及 inline-fallback 若发生时的 CP8 risk acceptance 处理策略。

该启动约束已在 CR151 中执行完毕，保留为未来 CR152/CR153 的流程参照。

### 4.6 建议交付对象

| 对象 | 目的 |
|---|---|
| `MultipleTestingReport` | 记录 candidate count、raw p-values、FDR/BH adjusted p-values、family definition、rejected hypotheses。 |
| `RobustFactorStatisticsReport` | 记录 IC / RankIC、Newey-West 或等价稳健 t-stat、IC decay、half-life、monotonicity、turnover。 |
| `WalkForwardValidationPlan` | 记录 train / validation / OOS windows、rolling policy、fold count、embargo assumptions。 |
| `BacktestOverfitRiskReport` | 记录 PBO / CSCV 或 staged approximation、DSR、trial_count、MinBTL / MinTRL inputs。 |
| `CapacityImpactReport` | 记录 ADV participation cap、capacity dollars、slippage / impact model family、turnover-capacity interaction。 |
| `BenchmarkRelativeReport` | 记录 IR、tracking error、Active Share、benchmark ref、factor / sector / style exposure summary。 |
| `FactorCorrelationClusterReport` | 记录因子 pairwise correlation、cluster assignment、redundant factor treatment。 |
| `StrategyAdmissionStatisticalGate` | 将上述报告收敛为 pass / fail / needs_review，并阻断未满足统计可靠性的 admission。 |

### 4.7 Exit Criteria

| 条件 | 门槛 |
|---|---|
| Contract availability | 100% 新增统计门对象有源码级 dataclass / typed dict / schema 或项目既有等价契约。 |
| Fixture coverage | 每个 gate 至少覆盖 PASS、FAIL、NEEDS_REVIEW 或 BLOCKED 中适用状态。 |
| Traceability | Strategy admission package 可追溯到统计报告 refs、BacktestRunSpec ref、ResearchDatasetSpec ref。 |
| Boundary proof | forbidden operations 仍为 0；真实 lake/NAS/provider/runtime/broker/credential 全部不触发。 |
| Release decision | CP8 只能在 static-only 口径下给出 READY / READY_WITH_RISK / NOT_READY，不得宣称 runtime 或真实收益可靠性。 |

## 5. ML 策略 E2E 框架整改计划（Future CR152）

### 5.1 目标

在现有 ResearchDatasetSpec / BacktestRunSpec 基础上补齐 ML 策略从 dataset snapshot、labeling、training split、model evaluation 到 admission 的方法论契约。

方法论来源：本计划采用 `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` 对 López de Prado AFML Ch.3/5/7/8/10/12、Bailey-López de Prado PBO/DSR 和机器学习监控实践的归纳作为初始来源；该枚举不是穷尽集合，CR152 CP2/CP3 可根据项目风险和可实现性裁剪或扩展。

### 5.2 分阶段整改

| 阶段 | 工作项 | 关键产物 |
|---|---|---|
| ML-1 Contract convergence | 复用现有 ResearchDatasetSpec / BacktestRunSpec，禁止重建平行 contract。 | ML extension plan、contract delta。 |
| ML-2 Purged + embargo CV | 将 purged k-fold、embargo、label overlap audit 作为 split policy。 | `PurgedEmbargoSplitPolicy`、split audit report。 |
| ML-3 Labeling | 支持 triple-barrier labeling 与 meta-labeling config。 | `TripleBarrierLabelSpec`、`MetaLabelingConfig`。 |
| ML-4 Feature importance | 输出 MDI / MDA / SHAP 或项目选定等价解释工件。 | `FeatureImportanceReport`。 |
| ML-5 Sample weighting | 处理样本唯一性、时间重叠、class imbalance 与 threshold tuning。 | sample uniqueness weights、class balance report。 |
| ML-6 Overfit gate | 记录 trial_count、PBO/DSR、MinBTL / MinTRL，纳入 admission。 | `MLOverfitRiskReport`。 |
| ML-7 Drift and retrain | 定义 PSI / KS / KL / Jensen-Shannon 等 drift 指标和 retrain trigger。 | `ModelDriftReport`、retrain decision record。 |
| ML-8 Artifact binding | 绑定 model artifact、training config、feature version、dataset snapshot、code version。 | ML artifact manifest。 |

### 5.3 初始边界

CR152 第一轮应以 contract + fixture + static validation 为主，不默认训练真实模型，不访问真实 provider，不写 lake，不启动 runtime。真实训练、模型注册、paper 或 live shadow 应作为后续 runtime authorization。

### 5.4 CP2 必须决策项

CR152 CP2 Decision Brief 应在常规 scope / implementation / security 决策之外，显式纳入以下两项：

| 决策 ID 建议 | 决策类型 | 待确认问题 | 推荐方向 | 影响 |
|---|---|---|---|---|
| `DQ-CP2-CR152-ML-GATE-RELATION` | architecture | ML admission gate 与 CR151 `StrategyAdmissionStatisticalGate` 的关系是什么？ | 新建 ML-specific admission gate，并通过 adapter 复用 CR151 四态 status 语义和 admission package linkage。 | 避免把 ML 专项方法硬塞进多因子 gate，同时保持三类策略 admission 状态一致。 |
| `DQ-CP2-CR152-STAT-GATE-OPT-IN` | implementation | UC-58 / UC-59 调用方何时将 statistical gate 从 opt-in 改成默认或强制？ | CR152 先保持 opt-in，但要求 ML/多因子 admission package 明确记录 gate_present / gate_required / gate_status；强制默认留给 CR154 或调用方治理。 | 避免悄悄改变历史默认行为，也防止新框架绕过 admission。 |

CP2 还应继续确认：

| 项 | 推荐边界 |
|---|---|
| validation mode | `static-only` / fixture-only |
| real model training | not authorized |
| real lake / NAS / provider / QMT / runtime / simulation / live / trading / broker / credential / external framework / Git remote | not authorized |
| first wave | PIT feature matrix、label policy / leakage guard、purged+embargo CV、training snapshot / model artifact metadata、prediction artifact、ML admission gate |

## 5.5 并行治理项（不阻塞 CR152）

以下治理债已累积，但不应阻塞 CR152 scope gate：

| 治理项 | 来源 | 建议处理 |
|---|---|---|
| STATE v2 hygiene | `CR151-CP8-R03-STATE-V2-HYGIENE` 已被接受为 process caveat。 | 单独 follow-up：迁出 `STATE.current.json.checkpoints` / `human_gate_decisions`，压缩 `STATE.current.json` 与 `STATE.md` 到 v2 限制。 |
| CR-INDEX legacy warnings | cr-tracking 仍有 legacy status / lifecycle 不一致和 follow-up candidate missing items。 | 单独 CR-index hygiene follow-up；不要混入 CR152 strategy scope。 |
| return / verify packet 字段非空校验 | CR151 多轮发现 actor / decision / design_delta / verification_plan 等字段可为空但 schema 通过。 | 加强 `meta-flow story return-check` 或新增 guardrail，校验关键字段非空。 |

## 6. 事件驱动策略 E2E 框架整改计划（Future CR153）

### 6.1 目标

补齐事件驱动策略从事件收集、PIT 修订、防泄漏、事件研究统计、信号生成、回测准入到 event-to-order trace 的方法论契约。

方法论来源：本计划采用 `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` 对 MacKinlay 事件研究、Brown-Warner 日频事件研究、Boehmer-Masumeci-Poulsen 事件引起方差、Lyon-Barber-Tsai 长窗收益、Kothari-Shanken-Sloan PIT 修订偏差和 White/Hansen 多重检验的归纳作为初始来源；该枚举不是穷尽集合，CR153 CP2/CP3 可按事件类型和数据可得性裁剪或扩展。

### 6.2 分阶段整改

| 阶段 | 工作项 | 关键产物 |
|---|---|---|
| EV-1 Event contract | 明确 event_time、available_at、decision_time、source reliability、revision policy。 | `EventResearchSpec`、event source quality report。 |
| EV-2 Event study method | 指定 estimation window、normal return model、event window、CAR / BHAR / calendar-time 方法。 | `EventStudyMethodSpec`。 |
| EV-3 Test family | 支持 BMP、Patell、generalized sign、rank、bootstrap 或项目选定检验族。 | `EventStudyTestReport`。 |
| EV-4 PIT revision gate | 防止使用事后修订数据、当前快照或不可交易时间。 | `EventRevisionPITGate`。 |
| EV-5 Overlap / clustering | 处理同日、同周、同主体重叠事件与聚类方差。 | event cluster report、cluster robust variance ref。 |
| EV-6 Endogeneity | 记录 self-selection、matching、PSM、IV 或 n/a-with-reason。 | endogeneity treatment note。 |
| EV-7 Admission gate | 将 FDR/BH、White/Hansen、PBO/DSR、capacity/impact、crowding/decay 纳入准入。 | `EventStrategyAdmissionGate`。 |
| EV-8 Event-to-order trace | 定义从 event decision 到 signal、target、paper order、broker order 的状态机；真实下单后置。 | trace state machine spec。 |

### 6.3 初始边界

CR153 第一轮不实现 live event listener，不对接 broker，不触发真实 event feed。事件数据可用 fixture 或静态样本模拟，重点验证方法论契约和 fail-closed gate。

## 7. 横切生产可靠性计划（Future CR154）

以下能力横跨多因子、ML 和事件驱动，不建议重复塞进单个策略 CR：

| 能力 | 目的 |
|---|---|
| Market impact model family | 将 commission / tax / simple slippage 扩展到 square-root、Almgren-Chriss style、implementation shortfall、TCA calibration。 |
| Capacity and liquidity sizing | 基于 ADV participation、fill probability、turnover、capacity dollars 阻断不可实现规模。 |
| Walk-forward / OOS framework | 三类策略统一的 rolling / purged / embargoed OOS 验证框架。 |
| Backtest trap gate | lookahead、survivorship、data snooping、regime overfit、cost underestimation 的统一检查层。 |
| Attribution | Brinson / factor attribution / strategy-specific attribution 的工件契约。 |
| Reconciliation | live-vs-offline、paper-vs-backtest、position/cash、intraday-vs-daily break workflow。 |
| Regime analysis | 牛熊、波动率、流动性、行业或 macro regime 条件化表现。 |

## 8. 数据湖任务后置规则

当前数据湖相关候选项不阻塞 CR151 / CR152 / CR153 的本地框架补齐：

| 项 | 后置原因 | 触发条件 |
|---|---|---|
| `RA-CR149-001` | 内容层已同步，strict metadata exit 未闭环但不影响本地策略契约设计。 | 需要宣称 NAS shared current truth strict PASS。 |
| `FU-CR149-002` | p/g parity 是 NAS filesystem governance，不是策略统计可靠性前置。 | 需要进入 Phase 5 readonly / paper 多节点 shared-read。 |
| `FU-CR139-001` | business-conflict groups 已在 quarantine / semantic-rule-required 状态，可治理但不要求马上解除。 | 需要解除 `prices` / `prices_limit` / `events` / `trade_status` quarantine 或推进生产级 turnover。 |
| `RA-CR139-002` | provider/NAS/delete/legacy archive 涉及高风险运行授权。 | 需要 provider refresh、NAS read/write 或 archive delete。 |
| `FU-CR140-001` | readiness=n/a，属于可选真实 lake readonly turnover semantic validation。 | 需要真实 lake 只读 turnover 语义验证作为 release blocker。 |

## 9. 推荐推进顺序

1. **CR152 ML Strategy E2E Framework Foundation**：下一正式推荐项。先走 CP0 / CP2，补 ML 专项 split、labeling、feature importance、sample uniqueness、drift、prediction artifact 和 ML admission gate。
2. **Hygiene follow-up（并行、不阻塞）**：STATE v2 slimming、CR-INDEX legacy warning、return / verify packet 字段非空校验。
3. **CR153 Event-Driven Strategy E2E Framework Foundation**：补事件研究方法论、PIT revision、event clustering、endogeneity 与 event admission gate。
4. **CR154 Cross-Strategy Production Reliability Gates**：统一 market impact、capacity、walk-forward、attribution、reconciliation、regime 分析，并决定 CR151/CR152 gate 的默认强制策略。
5. **Data lake production closure candidates**：策略框架需要生产级数据或多节点 shared-read 时，再恢复 RA-CR149-001 / FU-CR149-002 / FU-CR139-001 等候选项。

## 10. Gotchas

1. **不要把 metadata chain 当作统计可靠性。** CR150 证明链路完整，不证明因子、模型或事件策略具有样本外预测力。
2. **不要重复创建 ResearchDatasetSpec / BacktestRunSpec。** 当前源码已存在这两个契约；后续应扩展而不是平行实现。
3. **不要提前解除 quarantine。** FU-CR139-001 只有在生产级 turnover 或用户明确需要解除 quarantine 时才值得投入高成本逐 dataset 门禁。
4. **不要为了通过 NAS strict checker 修改 p/g 或 checker 语义。** FU-CR149-002 需要单独治理，不应混入策略框架补齐。
5. **不要让 ML 或事件驱动绕过 admission。** 三类策略都必须先通过统计、容量、成本、leakage 和 lifecycle gate。
6. **不要把 fixture PASS 宣称为真实收益证据。** CR151/152/153 第一轮的目标是契约和门控语义，不是生产策略收益证明。
7. **不要把 opt-in capability 误读为默认强制。** CR151 的 `require_statistical_gate=False` 是历史兼容设计；后续是否默认强制必须有 CP2/CP3 决策。
8. **不要让 hygiene 债吞掉策略框架主线。** STATE/CR-INDEX/tool-check 治理应并行小切片处理，不应扩大 CR152 的 ML scope。
