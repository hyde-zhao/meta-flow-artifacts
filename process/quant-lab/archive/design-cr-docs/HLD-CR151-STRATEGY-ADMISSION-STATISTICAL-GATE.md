---
status: "draft-for-cp3"
version: "1.1"
cr_id: "CR-151"
title: "Strategy Admission Statistical Gate HLD"
complexity: "standard"
selected_option: "dedicated metadata-only statistical gate contract module"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-01T23:24:00+08:00"
source_cp2: "process/checkpoints/CP2-CR151-MULTIFACTOR-STRATEGY-E2E-SCOPE.md"
discussion_log: "process/discussions/CP3-CR151-HLD-DISCUSSION-LOG.md"
discussion_checkpoint: "process/checks/CP3-CR151-DISCUSSION-CHECKPOINT.json"
authorization_boundary: "static/fixture/design only; no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote"
---

# HLD: CR151 Strategy Admission Statistical Gate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-01 | host-orchestrator | 初版：基于 CP2 approved scope、E2E review、roadmap 和现有源码契约，设计多因子统计准入门 Wave A。 |
| 1.1 | 2026-07-01 | host-orchestrator | 补齐评审追踪：显式 defer MF-GAP-2 非 Newey-West 评价统计、MF-GAP-4 regime 分层、MF-GAP-7 因子相关性聚类 / 去重；修正 Wave A 边界说明。 |

## 1. 问题定义

### 问题陈述

CR150 已证明多因子研究链路可以把 factor spec、factor run、factor panel、label window gate、signal set、portfolio policy、`BacktestRunSpec`、report pack、cost/risk attribution 和 `StrategyAdmissionPackage` 串成 metadata chain。这个链路仍不能回答“该多因子策略是否通过最低统计可靠性准入”。当前缺口集中在多重检验、稳健统计、样本外验证和回测过拟合风险：IC、RankIC 或 Sharpe 可以被参数搜索、数据窥探、自相关和样本内拟合放大。

CR151 的目标是补齐本地/static/fixture 多因子统计准入框架，让 admission 不只看链路完整，还必须看统计可靠性证据。

### 核心价值

用户获得一个可审计、可测试、可 fail-closed 的多因子 admission gate：策略报告即使链路完整，也会在缺少 FDR、稳健统计、walk-forward/OOS 或 PBO/DSR 证据时被阻断或标记 `needs_review`，避免把统计幻觉误认为可推进策略。

### 目标

| 优先级 | 目标 | 度量方式 |
|---|---|---|
| P0 | 新增 Wave A 统计准入对象 | 5 类对象在源码或等价 contract 中存在：multiple testing、robust statistics、walk-forward/OOS、overfit risk、statistical gate。 |
| P0 | Gate fail-closed | PASS、FAIL、NEEDS_REVIEW、BLOCKED 中至少 3 类状态被 fixture 覆盖；缺 mandatory evidence 时必须 BLOCKED。 |
| P0 | 复用既有契约 | `ResearchDatasetSpec`、`BacktestRunSpec`、`StrategyAdmissionPackage` 只被引用或扩展，不创建平行替代对象。 |
| P0 | 安全边界保持为零真实操作 | 禁止操作计数中 lake/NAS/provider/QMT/runtime/broker/credential/external framework/Git remote 均为 0。 |
| P1 | CR150 completion map 可追溯到统计 gate | `MultifactorFrameworkCompletionMap` 或后续 CR151 linkage 可引用 statistical gate report refs，且 `linkage_gaps=[]` 或明确 `statistical_gate_missing`。 |
| P2 | 为 CR152/CR153 留扩展点 | 新对象命名不硬编码到“multifactor only”，至少允许 future adapter 引用。 |

### 成功标准

- [ ] 100% Wave A objects 有源码级 contract 或等价 typed structure。
- [ ] 100% mandatory evidence missing path fail-closed。
- [ ] 100% CR151 tests 为 local fixture/static；不读取 `.env`、真实 lake、NAS、provider、QMT、broker 或外部框架。
- [ ] 1 个 admission gate summary 可被 strategy admission package 或 CR151 linkage map 引用。
- [ ] CP7/CP8 明确 `effective_validation_mode=static-only`，不得宣称 runtime 或真实收益可靠性。

### 约束

| 类型 | 约束内容 |
|---|---|
| 技术 | 复用当前 Python/uv/test stack；优先 stdlib/dataclass/项目现有 helpers；不得引入未授权外部框架。 |
| 业务 | 本 CR 不证明真实策略收益，只证明本地统计 admission semantics。 |
| 安全 | 不读 `.env`，不访问真实 lake/NAS/provider/QMT/broker，不做 runtime、simulation、paper、live 或 trading。 |
| 工作流 | CP3 通过前不得 Story decomposition；CP5 通过前不得实现；CP8 仍需 release readiness。 |

### 非目标

- 不实现真实数据湖读取、真实 factor panel 构建或 NAS 写入。
- 不实现 paper/live/simulation/trading/broker/account 查询。
- 不解除 FU-CR139-001 business-conflict quarantine。
- 不关闭 FU-CR149-002 NAS metadata parity。
- 不引入 Qlib、mlfinlab、pyfolio 或其他外部框架运行。
- 不把 fixture PASS 解释为真实收益可靠性。

### 关键假设

| 假设 | 若不成立的影响 |
---|---|
| Existing `ResearchDatasetSpec` and `BacktestRunSpec` are valid extension anchors. | 若源码契约不稳定，需先修复 CR147/CR148 contract hygiene，再回到 CR151。 |
| Wave A is sufficient for minimum statistical admission. | 若用户要求 production turnover/capacity，需把 Wave B/CR154 前置。 |
| Fixture/static validation can prove gate semantics. | 若需要 empirical proof，必须另起 real-data runtime authorization gate。 |

### 缺失信息

| 优先级 | 缺失信息 | 影响范围 | 处理 |
|---|---|---|---|
| BLOCKING | 无 | N/A | N/A |
| REQUIRED | 具体阈值默认值，例如 FDR alpha、minimum robust t-stat、DSR threshold、PBO threshold。 | CP5/implementation tests | CP3 推荐默认阈值，CP5 可按 Story 细化。 |
| OPTIONAL | 是否将 capacity/impact、IR/TE/Active Share、完整因子评价诊断、regime-aware validation 或因子相关性聚类 / 去重提前。 | Wave B/CR154 | 默认 deferred。 |

## 2. Architecture Gray Areas

**讨论日志**：`process/discussions/CP3-CR151-HLD-DISCUSSION-LOG.md`

**讨论恢复点**：`process/checks/CP3-CR151-DISCUSSION-CHECKPOINT.json`

| 灰区 ID | 关键问题 | 推荐结论 | 状态 |
|---|---|---|---|
| AGA-CR151-01 | 统计准入对象应独立成模块还是嵌入 mature runner？ | 独立 metadata-only contract module。 | selected |
| AGA-CR151-02 | CR151 是否一次性纳入 Wave B？ | Wave A only，Wave B 转 follow-up/CR154。 | selected |
| AGA-CR151-03 | fixture 不完整时如何定状态？ | `PASS / FAIL / NEEDS_REVIEW / BLOCKED`，mandatory missing 为 BLOCKED。 | selected |
| AGA-CR151-04 | 是否保持 static-only？ | 保持 static/fixture-only。 | selected |

### 方案形成输入

| 来源 | 影响章节 | 处理结果 |
|---|---|---|
| E2E review C1/C2/MF-GAP-1/2/4 | §4、§7、§9 | adopted for Wave A hard admission prerequisites; non-core evaluation diagnostics and regime split are explicitly deferred. |
| CP2 approved Wave A first | §3、§10 | adopted |
| Current source fact correction | §4、§5、§8 | adopted |
| Route A decision | §1、§11 | adopted |
| E2E review MF-GAP-7 | §2、§9、§11 | deferred to CR154 / follow-up because factor correlation clustering and redundancy de-duplication are portfolio construction diagnostics, not CR151 Wave A mandatory statistical gate inputs. |

### Deferred Architecture Ideas

| ID | 内容 | 延后原因 | 触发条件 |
|---|---|---|---|
| DAI-CR151-01 | Capacity / impact report | Cross-strategy production reliability; better in CR154. | 用户要求 production turnover/capital sizing。 |
| DAI-CR151-02 | Benchmark-relative IR/TE/Active Share | Evaluation/reporting artifact, not Wave A admission hard prerequisite. | CR154 或 CP3 scope expansion。 |
| DAI-CR151-03 | PIT universe audit | Intersects data-lake truth/quarantine governance. | 恢复真实数据湖或生产 turnover。 |
| DAI-CR151-04 | Extended factor evaluation diagnostics: IC decay by lag, half-life, turnover, liquidity/capacity view, orthogonalization against known risk factors, monotonicity and quantile spread | Important for full factor research quality, but Wave A only requires the minimum robust-statistical admission signal plus FDR/OOS/PBO-DSR. These diagnostics should be implemented as evaluation/reporting artifacts after the hard admission gate exists. | CR154, CR151 follow-up, or user requests production-grade factor evaluation before ML/event work. |
| DAI-CR151-05 | Regime-aware validation / regime-stratified backtest | MF-GAP-4 includes regime split, but CR151 Wave A only implements walk-forward/OOS fold evidence. Regime modeling needs explicit regime labels or real-data semantics and is better suited to CR154 / production reliability. | User requires policy-cycle, market-state or regime-specific admission before production turnover. |
| DAI-CR151-06 | Factor correlation clustering and redundancy de-duplication | MF-GAP-7 is a major portfolio construction and factor set governance item. It is not required to prove the first statistical admission gate semantics and should not be silently treated as covered by FDR/Newey-West/OOS/PBO-DSR. | CR154, a post-CR151 Story, or explicit CP3 scope expansion to include factor set de-duplication. |

## 3. 候选架构方案对比

### 方案 A：Dedicated Statistical Gate Contract Module

核心思路：新增一个 metadata-only statistical admission contract module，提供 report dataclasses、status evaluation、serialization 和 validation helpers；现有 `mature_multifactor_research` 和 `StrategyAdmissionPackage` 只消费 gate summary/ref。

| 维度 | 评估 |
|---|---|
| 优点 | 模块边界清晰；fixture 测试容易；CR152/CR153 可复用；不扩大 mature runner。 |
| 缺点 | 新增模块和 linkage Story。 |
| 复杂度 | medium |
| 可扩展性 | high |
| 风险 | 需要确保不和已有 `FactorModelValidationReport` 重复。 |
| 适用前提 | Gate 仍是 metadata/fixture contract，不运行真实数据。 |

### 方案 B：Embed in Mature Multifactor Runner

核心思路：直接在 `engine/mature_multifactor_research.py` 中增加 FDR/Newey-West/walk-forward/PBO/DSR report 和 gate builder。

| 维度 | 评估 |
|---|---|
| 优点 | 变更文件少，和 CR150 linkage 接近。 |
| 缺点 | 文件过宽；复用差；更难独立测试；后续 ML/event 会再造一套。 |
| 复杂度 | medium-low now, high later |
| 可扩展性 | low |
| 风险 | 容易把 runner output 与 admission policy 混合。 |
| 适用前提 | 只做临时小补丁，不追求 E2E framework completion。 |

### 方案 C：Extend StrategyAdmissionPackage Directly

核心思路：把统计报告字段和 gate 逻辑直接塞进 `engine/strategy_admission_package.py`。

| 维度 | 评估 |
|---|---|
| 优点 | Admission surface 集中。 |
| 缺点 | 现有 package 同时承担 no-real-operation/QMT blocked claims；直接放入统计计算会混淆研究准入与运行授权。 |
| 复杂度 | medium |
| 可扩展性 | medium |
| 风险 | 误把 statistical pass 解释为 simulation/live readiness。 |
| 适用前提 | 只增加 reference fields，不承载统计 gate 内部逻辑。 |

### 方案对比矩阵

| 维度 | 方案 A | 方案 B | 方案 C |
|---|---|---|---|
| 实现难度 | medium | low initially | medium |
| 可维护性 | high | low | medium |
| 可测试性 | high | medium | medium |
| 跨策略复用 | high | low | medium |
| 安全边界清晰度 | high | medium | medium-low |
| CP5 Story 切片 | clear | less clear | mixed |

**推荐方案**：方案 A。它以最小长期复杂度完成 CR151，同时不改写既有 admission/runtime 安全语义。

## 4. 推荐方案总览

**复杂度模式**：`standard`

| 判定维度 | 依据 | 结论 |
|---|---|---|
| 需求规模 | 5 类 Wave A objects + linkage + tests | standard |
| 状态流转 | Gate summary has 4 statuses | standard |
| 平台适配 | Local Python only | simple |
| Story 拆解 | 4-5 Stories likely | standard |

**核心思路**：

CR151 在现有研究/回测/准入链路上增加一个统计准入层。该层只消费已有 metadata refs 和 fixture numeric samples，输出 deterministic reports 和 gate summary。所有真实数据、运行和交易能力仍由后续 CR/gate 控制。

**关键架构风格**：layered metadata contract + fail-closed gate.

**核心能力边界**：

| 做 | 不做 |
|---|---|
| 定义 Wave A statistical reports | 不读真实 lake/NAS/provider |
| 评估 multiple testing / robust stats / OOS / overfit gate | 不运行真实 backtest或 simulation |
| 输出 pass/fail/needs_review/blocked summary | 不授权 QMT/broker/trading |
| 链接到现有 StrategyAdmissionPackage / CR150 completion map | 不替代 ResearchDatasetSpec / BacktestRunSpec |

## 5. 模块职责

| 模块 / 对象 | 责任 | 输入契约 | 输出契约 | 降级 / 失败 |
|---|---|---|---|---|
| `engine.strategy_admission_statistical_gate` | Wave A report contracts and gate evaluator | fixture metrics, report refs, thresholds, operation counters | `MultipleTestingReport`, `RobustFactorStatisticsReport`, `WalkForwardValidationPlan`, `BacktestOverfitRiskReport`, `StrategyAdmissionStatisticalGate` | missing mandatory evidence -> BLOCKED; threshold gray area -> NEEDS_REVIEW |
| `engine.mature_multifactor_research` | Build/link CR151 summary into existing completion chain | Existing CR150 nodes + statistical gate ref | updated linkage or builder helper | gate missing -> linkage gap, no runtime claim |
| `engine.strategy_admission_package` | Consume statistical gate reference and blocked/allowed claims | gate summary/ref | additional evidence refs or blocked reason | never converts statistical PASS to runtime authorization |
| `tests/test_cr151_strategy_admission_statistical_gate.py` | Fixture semantics coverage | local fixture samples | pass/fail/needs_review/blocked assertions | no external dependency |
| Process evidence | CP6/CP7/CP8 truth | return/evidence/result JSON | static-only evidence and release context | no runtime evidence claims |

## 6. 适用性矩阵

| 适用性维度 | 当前项目判断 | 推荐方案如何适配 | 不适配信号 | When to switch |
|---|---|---|---|---|
| 用户目标 | 先补齐本地策略框架 | Adds admission quality gate without data-lake work | 用户要求生产级 turnover | Switch to CR154/data-lake gates |
| 项目成熟度 | Existing research/backtest/admission contracts available | Reuses existing anchors | Existing contracts break tests | repair CR147/CR148 first |
| 认知负担 | Need clear audit chain | Dedicated reports map to E2E review gaps | Too many states confuse users | collapse display, not internal state |
| 验证条件 | fixture/static available | deterministic examples prove semantics | need empirical proof | separate runtime gate |
| 回退成本 | Low if module isolated | remove module/linkage tests only | embedded mutations spread | avoid方案 B |

### 优化 / 牺牲 / 切换条件

| 方案选择 | 优化了什么 | 牺牲了什么 | 接受理由 | 切换条件 |
|---|---|---|---|---|
| Dedicated statistical gate | maintainability, reuse, safety, testability | one extra module | CR151 is a framework completion item, not a one-off patch | If CP5 proves no cross-strategy reuse and Story count must be reduced |
| Wave A only | CP5/CP6/CP7 focus | defers capacity/benchmark/PIT audit, extended factor diagnostics, regime split and correlation clustering / de-dup | User chose local framework first; production closure later | If user changes route to production lake closure or requires full production-grade factor evaluation before CR152/CR153 |
| Static-only validation | safety and speed | no real performance proof | CP2 explicitly did not authorize runtime | Separate runtime authorization |

## 7. Data And Status Model

### Report Objects

| Object | Required Fields | Pass / Block Signals |
|---|---|---|
| `MultipleTestingReport` | `family_id`, `candidate_count`, `raw_p_values`, `adjusted_p_values`, `alpha`, `method`, `rejected_count` | BLOCKED if missing family/candidates; FAIL if no candidate survives required threshold. |
| `RobustFactorStatisticsReport` | `metric_id`, `sample_count`, `ic_mean`, `rank_ic_mean`, `robust_t_stat`, `p_value`, `autocorrelation_lags` | NEEDS_REVIEW if sample too small; FAIL if robust t/p-value below threshold. |
| `WalkForwardValidationPlan` | `folds`, `train_window`, `validation_window`, `oos_window`, `embargo_days`, `fold_metrics` | BLOCKED if no OOS fold; FAIL if fold pass rate below threshold. |
| `BacktestOverfitRiskReport` | `trial_count`, `pbo`, `dsr`, `observed_sharpe`, `skew`, `kurtosis`, `sample_length` | BLOCKED if trial_count missing; FAIL if PBO/DSR thresholds fail. |
| `StrategyAdmissionStatisticalGate` | `status`, `report_refs`, `blocked_reasons`, `needs_review_reasons`, `operation_counts` | Any mandatory BLOCKED -> BLOCKED; any hard FAIL -> FAIL; uncertain threshold -> NEEDS_REVIEW; all pass -> PASS. |

### Wave A Boundary For Review Gaps

| Review Gap | Wave A Treatment | Explicitly Deferred Items |
|---|---|---|
| MF-GAP-2 factor evaluation statistics | Covers minimum robust-statistical admission through IC / RankIC summary, autocorrelation lags and Newey-West or equivalent robust t/p-value. | IC decay by lag, half-life, turnover, liquidity/capacity view, orthogonalization against known risk factors, monotonicity and quantile spread are evaluation/reporting artifacts, not mandatory Wave A gate fields. |
| MF-GAP-4 validation stability | Covers walk-forward / OOS fold manifest and fold pass-rate gate. | Regime-aware validation and regime-stratified backtest are deferred because they require regime definitions and may intersect real-data / production reliability work. |
| MF-GAP-7 factor set redundancy | Not in Wave A hard admission gate. | Factor correlation clustering and redundancy de-duplication are deferred to CR154 / follow-up and must not be inferred from multiple-testing or robust-statistics coverage. |

### Status Rules

| Status | Meaning | Example |
|---|---|---|
| PASS | All mandatory reports pass configured thresholds. | FDR survives, robust t acceptable, OOS folds pass, PBO low, DSR high. |
| FAIL | Evidence exists and violates threshold. | PBO above max or DSR below min. |
| NEEDS_REVIEW | Evidence exists but requires human review or sample is borderline. | sample_count below preferred threshold but not empty. |
| BLOCKED | Mandatory evidence missing or forbidden operation counter nonzero. | no walk-forward folds or credential_read > 0. |

## 8. Key Flows

### Flow 1: Build Fixture Statistical Reports

1. Test fixture creates candidate p-values, IC/RankIC sequence, fold metrics and overfit-risk metrics.
2. Contract builders normalize fields and compute deterministic summary values where needed.
3. Validators emit report-level issues.
4. Gate evaluator aggregates report statuses.
5. Gate output is serialized as JSON-safe dict and referenced by linkage/evidence.

Failure path: missing candidate family, no OOS fold, missing trial count, or nonzero forbidden counter returns `BLOCKED`.

### Flow 2: Link Gate Into Existing Admission Chain

1. CR150 completion map or CR151 linkage builder receives statistical gate summary/ref.
2. The strategy admission package gets a statistical gate evidence ref and blocked claim if gate is not PASS.
3. Existing no-real-operation claims remain unchanged.
4. Runtime readiness claims remain blocked unless separately authorized.

Failure path: statistical gate missing creates `statistical_admission_gate_missing` linkage gap and does not mutate runtime authorization status.

### Flow 3: CP7 / CP8 Evidence

1. CP6 records implementation objects and tests.
2. CP7 validates report status coverage, fail-closed behavior, JSON/evidence/index consistency and static-only boundary.
3. CP8 release readiness states whether CR151 is READY, READY_WITH_RISK or NOT_READY under static-only validation.

Failure path: any runtime claim or forbidden operation counter nonzero blocks release readiness.

## 9. Use Case Traceability

| Use Case / Requirement | Support Module | Key Flow | Failure Path | Verification |
|---|---|---|---|---|
| UC-58 multifactor admission | Statistical gate module + StrategyAdmissionPackage linkage | Flow 1/2 | missing gate -> BLOCKED | fixture tests + CP7 evidence |
| CR150 metadata chain follow-up | CR151 linkage helper | Flow 2 | linkage gap if gate missing | completion map/linkage test |
| E2E review C1 multiple testing | `MultipleTestingReport` | Flow 1 | adjusted p-values fail -> FAIL | FDR fixture |
| E2E review C2 OOS | `WalkForwardValidationPlan` | Flow 1 | no OOS fold -> BLOCKED | fold fixture |
| MF-GAP-2 robust stats | `RobustFactorStatisticsReport` | Flow 1 | weak robust t -> FAIL/NEEDS_REVIEW | robust stats fixture |
| PBO/DSR blocker | `BacktestOverfitRiskReport` | Flow 1 | bad PBO/DSR -> FAIL | overfit fixture |
| MF-GAP-2 extended evaluation statistics | Deferred Wave B / CR154 | N/A in Wave A | not claimed as covered | deferred trace row only |
| MF-GAP-4 regime-aware validation | Deferred Wave B / CR154 | N/A in Wave A | not claimed as covered | deferred trace row only |
| MF-GAP-7 factor correlation clustering / redundancy de-duplication | Deferred Wave B / CR154 | N/A in Wave A | not claimed as covered | deferred trace row only |

## 10. Story Slice Recommendation

| Story ID | Title | Design Evidence | Primary Files | Notes |
|---|---|---|---|---|
| CR151-S01 | Statistical report contracts | full-lld | `engine/strategy_admission_statistical_gate.py`, tests | Defines reports, status enum, validators. |
| CR151-S02 | Gate evaluator and fail-closed rules | full-lld | same module + tests | Covers PASS/FAIL/NEEDS_REVIEW/BLOCKED. |
| CR151-S03 | Linkage to existing admission/completion chain | full-lld | `engine/mature_multifactor_research.py`, maybe `engine/strategy_admission_package.py`, tests | Minimal linkage only; no runtime semantics change. |
| CR151-S04 | Process evidence and release wording | technical-note | process/evidence, CP6/CP7/CP8 docs | Static-only evidence and risk wording. |

CP4 may merge S01/S02 if CP5 shows file ownership is identical and Story size remains manageable. S03 should stay separate because it touches existing CR150 files.

## 11. Risks And Mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| Statistical PASS misread as live readiness | high | Blocked runtime claims remain in StrategyAdmissionPackage; CP8 says static-only. |
| Duplicating existing FactorModelValidationReport | medium | CR151 gate consumes or complements existing validation report; do not recreate current fields without need. |
| Wave B scope creep | medium | CP3 explicitly defers capacity/impact, benchmark-relative, PIT universe audit, extended factor diagnostics, regime-aware validation and factor correlation clustering / de-duplication. |
| Threshold arguments grow too large | medium | Use configurable default thresholds; keep CP5 focused on semantics. |
| External framework temptation | low | Out of scope; future Spike only. |

## 12. NFR And Verification

| NFR | Design Response | Verification |
|---|---|---|
| Determinism | Pure local fixture inputs; JSON-safe serialization. | Repeat tests produce same dict/hash. |
| Safety | No real IO, no credentials, no provider/runtime. | Static guard and operation counters. |
| Maintainability | Dedicated module and report objects. | Unit tests per report and gate. |
| Traceability | report refs included in gate summary and admission linkage. | CP7 trace matrix. |
| Fail-closed | Missing mandatory evidence is BLOCKED. | negative fixture tests. |

## 13. HLD Split Assessment

| Signal | Assessment |
|---|---|
| Core product count | 1: statistical admission gate for strategy research. |
| Story count | 4 recommended, below split threshold. |
| ADR clustering | Single cluster: module boundary, status model, validation boundary. |
| Cross-layer risk | Runtime/security boundary is a constraint, not a separate product. |

Decision: keep one companion HLD for CR151. CR152/CR153/CR154 remain future CRs and should receive separate HLDs when activated.

## 14. CP3 Decision Inputs

| Decision ID | Type | Recommendation |
|---|---|---|
| DQ-CP3-CR151-001 | architecture | Approve dedicated metadata-only statistical gate contract module. |
| DQ-CP3-CR151-002 | scope | Approve Wave A only and defer Wave B to follow-up / CR154. |
| DQ-CP3-CR151-003 | architecture | Approve four-state fail-closed gate model. |
| DQ-CP3-CR151-004 | security | Keep static/fixture-only validation and no-runtime authorization boundary. |

## 15. Gotchas

1. Do not treat `StrategyAdmissionStatisticalGate.status=PASS` as simulation, paper, live or trading readiness.
2. Do not use fixture p-values or Sharpe values as real strategy evidence.
3. Do not re-create `ResearchDatasetSpec` or `BacktestRunSpec`; current source already has them.
4. Do not hide missing OOS or trial-count evidence as `NEEDS_REVIEW`; mandatory missing evidence is `BLOCKED`.
5. Do not pull Wave B capacity/impact/PIT audit into CR151 unless CP3 scope is explicitly changed.
6. Do not claim MF-GAP-2 is fully covered by Newey-West alone; IC decay, half-life, turnover, orthogonalization, monotonicity and quantile spread remain deferred evaluation artifacts.
7. Do not claim walk-forward coverage implies regime-aware validation; regime-stratified admission remains deferred.
8. Do not treat FDR or robust statistics as factor correlation clustering / redundancy de-duplication; MF-GAP-7 remains a follow-up / CR154 item.
