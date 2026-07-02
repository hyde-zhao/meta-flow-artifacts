---
status: "draft-current-index"
version: "1.7"
feature_id: "FEAT-03"
---

# Feature Test Plan: factor-research-loop

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-06-27 | codex | 增补 FactorCatalog / CLI 和 mature runner 因子纳管测试入口。 |
| 1.2 | 2026-06-27 | codex | 增补高级多因子模型评估、政策周期配置合同和 mature admission 集成测试入口。 |
| 1.3 | 2026-06-27 | codex | 增补 Stage 3 通过型候选门禁测试：blocked 候选不得关闭 Stage 3 或进入 Stage 4。 |
| 1.4 | 2026-06-27 | codex | 增补异象发现 / 研究测试入口：候选先验逻辑、Harvey 标准、单调性、控制因子、时间切分、成本和经济逻辑门禁。 |
| 1.5 | 2026-06-27 | codex | 增补自动异象发现测试：受控模板、批量 runner、多重检验、动态因子目录接入和 Stage 3 候选 FactorSpec。 |
| 1.6 | 2026-06-28 | codex | 增补研究引擎稳定模块和脚本归档测试：测试 import 指向领域名模块，质量检查禁止新增不稳定 engine / root script 主入口。 |
| 1.7 | 2026-07-01 | host-orchestrator | CR151 增补统计准入门测试入口：FDR、robust stats、walk-forward/OOS、PBO/DSR、fail-closed 状态、CR150 linkage 和 static-only 证据边界。 |

## 测试矩阵

| 测试域 | 关键场景 | 验证入口 |
|---|---|---|
| 外部参考边界 | 外部项目仅静态参考，不运行、不迁移源码 | `tests/test_cr030_external_reference_guardrails.py` |
| FactorSpec / RunSpec | schema、错误码、failure policy 完整 | `tests/test_cr030_factor_spec_run_spec_contract.py` |
| FactorCatalog / CLI | factor_id 唯一、状态枚举固定、核心 7 / mature runner 5 / anomaly proxy 3 覆盖、CLI table/json/filter 可用、未知 factor_id fail-closed | `tests/test_factor_registry.py`、`tests/test_list_factors_cli.py` |
| Mature runner 因子纳管 | Mature runner `FactorSpec` 必须从统一目录读取元数据，runner 5 因子必须是 `stage3_active` 兼容状态且声明 runner-local calculator | `tests/test_stage3_mature_multifactor_research.py` |
| 异象发现候选 | anomaly proxy 必须通过 `engine.anomaly_research` 生成 AnomalyCandidate；每个候选必须有 source_type、hypothesis、economic_rationale、prior_logic_ref、expected_direction 和 required_factor_ids；无先验逻辑时 fail-closed | `tests/test_chapter5_anomalies.py` |
| 异象研究报告 | 必须报告 High-Low 单变量排序、Harvey t>=3、五分组单调性、控制因子 alpha、时间切分、扣费后收益、A 股特殊处理和经济故事状态 | `tests/test_chapter5_anomalies.py` |
| 异象准入 | t<3、分组单调失败、alpha 控制失败、时间切分失败、扣费后不可行或经济故事缺失时不得输出 `factor_catalog_candidate` | `tests/test_chapter5_anomalies.py` |
| 自动候选生成 | 只能从受控模板生成候选；字段缺失时跳过；候选必须记录 formula、transform、hypothesis、economic_rationale 和 prior_logic_ref | `tests/test_anomaly_candidate_generator.py` |
| 自动 discovery runner | 批量输出候选、研究报告、准入决策、candidate_count、多重检验字段和 no-real-op counters；label leakage fail-closed | `tests/test_anomaly_discovery.py` |
| 自动准入 / 目录接入 | 通过多重检验的候选可作为动态 FactorCatalogEntry extras 查询；未通过项不得成为 `stage3_candidate`；静态 registry 不被改写 | `tests/test_anomaly_discovery.py`、`tests/test_factor_registry.py` |
| Stage 3 anomaly candidate spec | Stage 3 candidate spec 构建可显式消费动态 `stage3_candidate`，默认固定 5 因子顺序不变 | `tests/test_stage3_mature_multifactor_research.py` |
| 高级多因子模型评估 | GRS、因子溢价、经济显著性、时间切分、测试资产多样性、样本外、壳价值、做空可行性、政策周期和稳健性评估生成统一报告；核心门禁 blocked 时 overall blocked | `tests/test_factor_model_validation_report.py` |
| 政策周期配置合同 | 内置政策周期配置可加载；覆盖率计算必须显式记录 covered / missing cycles 和 dataset candidates | `tests/test_policy_cycle_dataset_contract.py` |
| Mature runner 高级评估集成 | Mature runner 生成 `FACTOR-MODEL-VALIDATION-REPORT.json`，evidence refs 和 mature admission package 必须包含报告引用、状态和 gate summary | `tests/test_stage3_factor_model_validation_integration.py`、`tests/test_stage3_mature_multifactor_research.py` |
| CR151 statistical report contracts | MultipleTestingReport、RobustFactorStatisticsReport、WalkForwardValidationPlan、BacktestOverfitRiskReport 和 StrategyAdmissionStatisticalGate 可 JSON-safe 序列化；字段缺失返回可审计 issue | `tests/test_cr151_strategy_admission_statistical_gate.py` |
| CR151 fail-closed gate evaluator | PASS / FAIL / NEEDS_REVIEW / BLOCKED 四态均由 fixture 覆盖；mandatory evidence missing 与 forbidden operation counter nonzero 必须 BLOCKED | `tests/test_cr151_strategy_admission_statistical_gate.py` |
| CR151 linkage to admission package / completion map | statistical gate ref 可接入 StrategyAdmissionPackage 或 CR151 linkage helper；gate missing 产生 `statistical_admission_gate_missing`，不得生成 runtime readiness claim | `tests/test_cr151_strategy_admission_statistical_gate.py`、`tests/test_cr150_multifactor_framework_completion.py` |
| CR151 static-only evidence boundary | 测试不得读取 `.env`、真实 lake/NAS/provider/QMT/broker、外部框架或 Git remote；no-real-operation counters 为 0 | `tests/test_cr151_strategy_admission_statistical_gate.py`、CP7 evidence scan |
| Stage 3 pass gate | 只有 FactorModelValidationReport 核心门禁无 blocked、mature admission PASS、runner offline / preflight PASS 且 no-real-op 计数为 0 的候选才能作为 Stage 3 输出；blocked 候选必须保持研究失败 / 诊断状态 | `tests/test_stage3_factor_model_validation_integration.py`、后续 `tests/test_stage3_candidate_pass_gate.py` |
| 候选搜索 / sweep | 多组因子、权重、样本切分、持仓数、换手和成本参数搜索时，每个候选都必须生成评估报告；只有通过候选可被标记为 stage3_pass_candidate | 后续 `tests/test_stage3_candidate_sweep.py` |
| FactorPanel / LabelWindow | 四层值、coverage、label window fail-closed | `tests/test_cr030_factor_panel_label_window_gates.py` |
| 因子评价报告 | IC、RankIC、分层收益、成本敏感性、blocked claims | `tests/test_cr030_factor_evaluation_report.py` |
| 多因子组合 | 权重、约束、benchmark、成本、冻结策略 | `tests/test_cr030_multifactor_combiner.py` |
| Manifest / Catalog | config hash、dataset release、report path、复跑证据 | `tests/test_cr030_experiment_manifest_catalog.py` |
| StrategyAdmissionPackage | 不等同交易授权，no-real-op 计数保持 0 | `tests/test_cr030_strategy_admission_package.py`、`tests/test_cr030_no_real_operation_safety.py` |
| 研究引擎稳定模块 | 旧 `engine/chapter*` 不得留在 `engine/` 顶层；测试 import 指向 `factor_replication`、`research_data_readiness`、`factor_model_research`、`anomaly_research`、`factor_robustness`、`factor_portfolio_practice`；旧 engine/root 实现集中归档 | `tests/test_script_entrypoint_naming.py`、相关 factor research tests |

## 手工验证

| 场景 | 预期 |
|---|---|
| 研究报告用于 QMT 后续评审 | 只能提供 evidence / blocked reason / order_intent_draft_ref，不触发 QMT |
| 外部项目分析更新 | 必须记录 reference / optional Spike / exclude / forbidden migration 分类 |
| Stage 3 blocked 运行复盘 | 明确 blocked 原因、修复方向和下一轮候选配置；不得把 blocked 运行写入 Stage 4 observation 输入 |
| CR151 release wording | 明确 `effective_validation_mode=static-only`，statistical PASS 不代表 simulation / paper / live / trading readiness。 |
