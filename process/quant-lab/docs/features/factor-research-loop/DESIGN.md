---
status: "draft-current-index"
version: "1.8"
feature_id: "FEAT-03"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md"
source_blueprint: "docs/design/BLUEPRINT.md"
change: "CR-031"
---

# Feature Design: factor-research-loop

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-07 | meta-po | 新增多因子研究闭环 Feature 设计索引 |
| 1.1 | 2026-06-23 | host-orchestrator | CR129 增加与 `strategy-runner-core` 的边界说明。 |
| 1.2 | 2026-06-27 | codex | 增补统一因子目录、查询 CLI 和 Stage 3 因子纳管；目录覆盖核心 7 因子、Stage 3 5 因子和 Chapter5 anomaly proxy 3 因子。 |
| 1.3 | 2026-06-27 | codex | 增补高级多因子模型评估报告、mature admission 门禁、政策周期配置合同和 no-real-op 边界。 |
| 1.4 | 2026-06-27 | codex | 收紧 Stage 3 目标：必须输出评估通过的成熟多因子策略；当前 blocked 运行只作为诊断基线和下一轮研究输入。 |
| 1.5 | 2026-06-27 | codex | 增补异象发现与异象研究方法：理论驱动候选、Harvey t>=3、五分组单调性、控制因子 alpha、时间切分、A 股可行性和经济逻辑准入。 |
| 1.6 | 2026-06-27 | codex | 增补自动异象发现系统：受控模板候选生成、批量研究 runner、多重检验、准入报告、动态因子目录接入和 Stage 3 候选消费。 |
| 1.7 | 2026-06-28 | codex | 增补研究引擎稳定模块整改：chapter/stage/root 旧入口归档到 legacy archive，领域名模块和共享 helper 成为主实现面。 |
| 1.8 | 2026-07-01 | host-orchestrator | CR151 增补 Strategy Admission Statistical Gate：Wave A 统计准入合同、fail-closed 四态门、CR150 linkage 和 static-only evidence；显式 deferred Wave B 评价工件。 |

## Feature 摘要

| 项 | 内容 |
|---|---|
| Feature 目标 | 统一项目自有多因子研究闭环，覆盖 FactorCatalogEntry、AnomalyCandidate、AnomalyDiscoveryRun、AnomalyResearchReport、AnomalyAdmissionDecision、FactorSpec、FactorRunSpec、FactorPanel、LabelWindow、FactorEvaluationReport、FactorModelValidationReport、MultiFactorCombiner、ExperimentManifest、StrategyAdmissionPackage，并支撑 Stage 3 输出评估通过的成熟多因子策略 |
| Owner | FEAT-03 |
| 主要代码面 | `engine/factor_*`、`engine/anomaly_*`、`engine/research_*`、`engine/multifactor_*`、`engine/serialization.py`、`engine/factor_research_matrices.py`、`engine/admission_contracts.py`、`scripts/list_factors.py`、`scripts/research/*`、`experiments/*factor*` |
| 主要设计来源 | `process/HLD.md` §35、ADR-079..086、CR-030 Story / LLD |
| 非授权声明 | 不授权外部项目运行、依赖变更、provider/lake/publish、QMT/simulation/live 或凭据读取 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 |
|---|---|---|---|
| ResearchDataset | 消费 `research_input_v1` 和 production current truth | 写湖 / publish | FEAT-02 |
| FactorSpec / FactorRunSpec | 因子定义、运行配置、错误码和 failure policy | 外部框架默认 schema | FEAT-04 |
| FactorCatalogEntry / FactorAvailabilityStatus / FactorUsageRef | 统一登记因子身份、来源、状态、计算能力、使用方和证据引用；提供只读 CLI 查询入口 | 数据湖读取、catalog publish、mature runner-local 计算逻辑迁移 | FEAT-13 / FEAT-14 |
| AnomalyCandidate / AnomalyResearchReport / AnomalyAdmissionDecision | 理论驱动候选生成、异象研究报告、异象准入和 factor catalog 候选动作 | 纯数据挖掘自动发现、真实事件数据生产、QMT / simulation / live 授权 | FEAT-10 / FEAT-14 |
| ControlledAnomalyTemplate / AnomalyDiscoveryRun | 受控自动候选模板、批量研究运行、搜索空间审计、多重检验和动态目录候选输出 | 黑箱公式搜索、从收益反推阈值、静态 registry 自动改写、catalog publish | FEAT-13 / FEAT-14 |
| FactorPanel / LabelWindow | 因子面板四层值、标签窗口、泄漏 fail-closed | 原始数据生产 | FEAT-02 |
| Evaluation / Combiner | IC、RankIC、分层收益、多因子组合、高级模型评估和报告 | 实盘执行、broker shortability facts 生产 | FEAT-06 |
| StrategyAdmissionPackage | 准入证据、FactorModelValidationReport 引用、blocked reasons、order intent draft 引用 | runtime authorization | FEAT-07 / FEAT-14 |
| StrategyAdmissionStatisticalGate | Wave A 统计准入证据、多重检验、稳健统计、walk-forward/OOS、PBO/DSR 和四态 fail-closed 汇总 | 真实收益证明、生产容量/冲击、regime 建模、交易授权 | FEAT-07 / FEAT-08 |

## 与 Strategy Runner Core 的边界

| 项 | 边界 |
|---|---|
| 本 Feature 负责 | 产出研究侧 factor evaluation、multifactor portfolio plan、experiment manifest、research report catalog 和 StrategyAdmissionPackage。 |
| `strategy-runner-core` 负责 | 将已形成的本地 strategy package / adapter payload 跑过 offline runner core，并输出 `RunResult` 与脱敏 evidence。 |
| 禁止误读 | StrategyAdmissionPackage 不等于 runner package pass，不等于 QMT-ready，也不授权 runtime / NAS / provider / lake / catalog / trading。 |

## 输入 / 输出契约

| 方向 | 契约 |
|---|---|
| 输入 | published data release、research_input_v1、factor definitions、label window、cost config、benchmark policy |
| 输出 | factor catalog、factor panel、factor evaluation report、factor model validation report、multifactor portfolio plan、experiment manifest、research report catalog、strategy admission package |
| 错误输出 | `schema_invalid`、`label_window_leakage`、`required_missing`、`blocked_claims`、`admission_blocked` |

## 因子目录与查询入口

| 项 | 设计 |
|---|---|
| 目录范围 | 核心 7 因子、mature runner 5 因子、异象代理 3 因子先纳入 `engine.factor_registry`。 |
| 最小字段 | `factor_id`、`name`、`category`、`family`、`direction`、`input_fields`、`formula`、`source_refs`、`status`、`calculator_status`、`used_by`、`evidence_refs`、`notes`。 |
| 状态枚举 | `defined`、`calculable`、`stage3_active`、`proxy_only`、`deprecated`、`blocked`。 |
| CLI | `scripts/list_factors.py` 只读代码内注册表，支持 table / json、status、used-by 和 factor-id 过滤。 |
| Mature runner | Stage 3 当前兼容 runner 的 `FactorSpec` 元数据优先从目录读取；runner-local 计算已迁移到领域名模块 `engine/mature_multifactor_research.py`，旧 `engine/stage3_mature_multifactor_research.py` 已归档到 `docs/legacy/archive/engine/`。 |

## 异象发现与研究

| 阶段 | 方法 | 当前工程门禁 |
|---|---|---|
| 候选变量生成 | 来自经济 / 行为理论、市场微观结构、财务指标扩展或事件 / 政策假设；必须记录 `hypothesis`、`economic_rationale`、`expected_direction`、`input_fields` 和 `prior_logic_ref`。 | 无先验逻辑的纯数据挖掘候选不得进入 `AnomalyResearchReport` pass。 |
| 初步筛选 | 单变量排序 High-Low、多空收益均值、Newey-West t 值、五分组单调性。 | Harvey t>=3 是升级底线；分组单调性必须报告，失败只能 watch / reject。 |
| 严格验证 | 控制既有因子模型后的 alpha、前后半期时间切分、子样本 / 市值 / 行业、交易成本、年化换手、A 股做空限制、壳价值和政策周期覆盖。 | 缺项必须进入 `gap_register` / risk warning；控制因子 alpha 或时间切分失败不得成为 `factor_catalog_candidate`。 |
| 经济逻辑解释 | 风险故事或行为故事；说明为什么不是数学巧合。 | 只有统计显著但无经济解释的异象标记 `statistical_artifact_risk`，不得进入 Stage 3 因子候选。 |

当前实现重点放在异象复刻与可执行研究门禁：`engine.anomaly_research` 负责将固定 anomaly proxy 扩展为 `AnomalyCandidate` 和 `AnomalyResearchReport`；旧 `engine/chapter5_anomalies.py` 已归档到 `docs/legacy/archive/engine/`。完整自动化大规模变量挖掘必须通过受控 candidate search，不允许跳过先验逻辑约束。

## 自动异象发现系统

| 层 | 设计 |
|---|---|
| 候选生成 | `engine.anomaly_candidate_generator` 只从 `ControlledAnomalyTemplate` 生成候选；默认模板覆盖估值、动量、反转、流动性、波动率、质量和投资类字段，字段缺失时跳过，不从收益结果反推公式。 |
| 批量研究 | `engine.anomaly_discovery` 消费本地 feature panel 和 model returns，复用 `engine.anomaly_research` 的排序收益、五分组单调、控制因子 alpha、时间切分、扣费后收益和经济逻辑报告。 |
| 多重检验 | `engine.anomaly_multiple_testing` 为每个候选记录本轮 `candidate_count`、raw p-value、Bonferroni p-value、BH-FDR q-value 和 `multiple_testing_pass`。 |
| 准入 | `engine.anomaly_admission` 将研究报告转为 `rejected`、`blocked_insufficient_data`、`research_candidate`、`factor_catalog_candidate` 或 `stage3_candidate`；未通过多重检验不得成为 `stage3_candidate`。 |
| 报告 | `scripts/research/run_anomaly_discovery.py` 输出 `anomaly_candidates.json`、`anomaly_research_report.json`、`anomaly_admission_decisions.json`、parquet 明细和 markdown 摘要。 |
| 因子目录 | `engine.factor_registry.anomaly_candidate_catalog_entries` 只把通过准入的候选作为动态 extras 显式接入，不修改静态 registry，不 publish catalog。 |
| Stage 3 | `engine.mature_multifactor_research.build_admitted_anomaly_factor_specs` 可显式消费 `stage3_candidate` 动态目录候选供后续 candidate search；默认成熟 runner 的固定 5 因子不变。 |

## 高级多因子模型评估

| 项 | 设计 |
|---|---|
| 报告 schema | `factor_model_validation_report_v1`，由 `engine.factor_model_validation` 生成，mature multifactor runner 输出 `FACTOR-MODEL-VALIDATION-REPORT.json`。 |
| 必选评估 | GRS 检验、因子溢价显著性、经济显著性、时间切分、测试资产多样性、样本外验证、壳价值控制、做空可行性、政策周期覆盖。 |
| 补充稳健性 | 因子冗余 / 相关性、风格暴露、参数敏感性、IC 衰减、多重检验控制、数据偏差审计、组合构建稳健性、容量冲击、尾部风险。 |
| 核心门禁 | 因子溢价显著性、经济显著性、样本外验证、数据偏差审计；任一 blocked 时报告 overall `blocked`，mature admission blocked。 |
| 非核心缺口 | GRS 因缺少 factor return / asset return、做空可行性因 long-only、政策周期覆盖因配置缺失等情况必须返回 `not_applicable` / `insufficient_data` / risk warning，不得伪造 pass。 |
| 政策周期 | 默认只读 `config/policy_cycles.yaml` 配置；真实政策周期数据集生产属于 FEAT-02 后续能力。 |
| 安全边界 | 报告只读输入面板、标签、组合、universe / tradability 摘要和配置；provider_fetch、lake_write、catalog_publish、qmt_api_call、real_order、credential_read 计数必须为 0。 |

## Strategy Admission Statistical Gate（CR151）

| 层 | 设计 |
|---|---|
| 统计准入目标 | 在既有多因子 metadata chain 上补最小可审计统计准入门，避免把链路完整、样本内 Sharpe 或单一 IC 误读为可靠策略。 |
| 新增合同面 | `engine.strategy_admission_statistical_gate`，包含 `MultipleTestingReport`、`RobustFactorStatisticsReport`、`WalkForwardValidationPlan`、`BacktestOverfitRiskReport` 和 `StrategyAdmissionStatisticalGate`。 |
| Wave A 必做 | FDR / multiple testing、Newey-West or equivalent robust t/p-value、walk-forward / OOS fold manifest、PBO / DSR、aggregate statistical gate status。 |
| 状态模型 | `PASS`、`FAIL`、`NEEDS_REVIEW`、`BLOCKED`；mandatory evidence missing 和 forbidden operation counter nonzero 必须 `BLOCKED`。 |
| Linkage | `engine.mature_multifactor_research` / `StrategyAdmissionPackage` 只消费 statistical gate refs、status 和 blocked reasons，不把 statistical PASS 转成 runtime readiness。 |
| Evidence | CP6/CP7/CP8 只声明 `effective_validation_mode=static-only`；fixture PASS 不代表真实 lake、NAS、provider、QMT、simulation、live、broker 或 trading 证据。 |
| Deferred Wave B | IC decay by lag、half-life、turnover、liquidity/capacity view、orthogonalization、monotonicity、quantile spread、regime-aware validation、factor correlation clustering / redundancy de-duplication、capacity / impact、IR/TE/Active Share、PIT universe audit。 |

## 研究引擎稳定模块

| 层 | 稳定入口 | 兼容 / 归档策略 |
|---|---|---|
| 因子复刻 | `engine.factor_replication` | 旧 `engine/chapter3_factor_replication.py` 已归档到 `docs/legacy/archive/engine/` |
| 数据 readiness | `engine.research_data_readiness` | 旧 `engine/chapter3_real_data_readiness.py` 已归档到 `docs/legacy/archive/engine/` |
| 因子模型研究 | `engine.factor_model_research` | 旧 `engine/chapter4_factor_models.py` 已归档到 `docs/legacy/archive/engine/` |
| 异象研究 | `engine.anomaly_research` | 旧 `engine/chapter5_anomalies.py` 已归档到 `docs/legacy/archive/engine/` |
| 稳健性研究 | `engine.factor_robustness` | 旧 `engine/chapter6_factor_robustness.py` 已归档到 `docs/legacy/archive/engine/` |
| 组合实践 | `engine.factor_portfolio_practice` | 旧 `engine/chapter7_factor_practice.py` 已归档到 `docs/legacy/archive/engine/` |
| 共享 helper | `engine.serialization`、`engine.factor_research_matrices`、`engine.admission_contracts` | 新增研究模块必须复用，不再私有新增同类 helper |
| 脚本入口 | `scripts/research/*`、`scripts/qmt/*` | 旧 root 实现归档到 `scripts/legacy/*`；root 只保留项目级稳定工具 |

## Stage 3 出口标准

| 项 | 必须满足 |
|---|---|
| 策略目标 | 输出一条基于真实数据湖、真实 PIT universe 和登记因子的成熟多因子策略，而不是仅输出研究运行包。 |
| 评估状态 | `FactorModelValidationReport.overall_status` 不得为 `blocked` / `insufficient_data`；核心门禁因子溢价显著性、经济显著性、样本外验证和数据偏差审计必须无 blocked。 |
| 准入状态 | mature StrategyAdmissionPackage 必须为 `PASS`，并引用评估报告、gate summary、risk warnings、runner offline preflight 和 no-real-op counters。 |
| runner 证据 | offline / plan-only / preflight-only 证据必须通过；仍不授权 gateway、QMT、simulation、live 或账户操作。 |
| 失败候选 | 当前 `stage3-mature-mf-20260627-real-lake-v4-validation-gated` 因 economic_significance 和 out_of_sample_validation blocked，只能作为诊断基线，不能关闭 Stage 3 或进入 Stage 4。 |

## 失败路径

| 失败点 | 行为 |
|---|---|
| 数据或 benchmark 缺失 | 生成 `required_missing`，不得输出 production admission pass |
| label window 泄漏风险 | fail-closed，不继续评价 |
| 外部项目只能静态参考 | clone / install / run / qrun / notebook / provider 下载均 blocked |
| admission package incomplete | 输出 blocked reason，不进入 QMT stage |
| Mature runner 或章节脚本引用未登记因子 | fail-closed，先补 FactorCatalogEntry 和测试，再允许消费 |
| 新增 chapter / stage / cr 命名的 engine 主实现 | quality guardrail fail；改用领域名模块或 legacy wrapper |
| FactorModelValidationReport 核心门禁 blocked 或报告缺失 | Mature admission blocked；报告路径必须进入 evidence refs |
| StrategyAdmissionStatisticalGate mandatory evidence 缺失 | statistical admission `BLOCKED`；不得用 `NEEDS_REVIEW` 隐藏缺失报告 |
| Statistical gate forbidden operation counter 非 0 | statistical admission `BLOCKED`；CP7/CP8 不得声明 runtime evidence |
| 高级评估缺少外部事实 | 返回 unavailable / risk warning，不反向写湖、不读取 broker 或 QMT |
| Stage 3 候选评估未通过 | 保留研究证据和 blocked reasons，回到因子、组合、样本切分、成本、风险约束或数据补齐迭代，不得进入 Stage 4 |

## Gotchas

- StrategyAdmissionPackage 是准入证据，不是交易授权。
- CR-030 不从零发明 schema，必须回链已有 `research_input_v1`、实验 17-21 和 CR-011 审计合同。
- Qlib / Alphalens / Zipline / LEAN 只能 cross-check，不成为 truth。
- CR151 statistical gate PASS 只代表本地 fixture/static 统计准入语义通过，不代表 production turnover、simulation、paper、live 或 QMT-ready。
- 不得把 MF-GAP-2 扩展评价统计、MF-GAP-4 regime 分层或 MF-GAP-7 因子相关性聚类 / 去重视为 CR151 已完成；这些仍是 CR154 / follow-up。
