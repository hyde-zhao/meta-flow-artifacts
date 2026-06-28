---
status: "draft-current-index"
version: "1.6"
feature_id: "FEAT-03"
---

# Feature Tasks: factor-research-loop

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-06-27 | codex | 增补统一因子目录、查询 CLI 和 mature runner 因子纳管任务。 |
| 1.2 | 2026-06-27 | codex | 增补高级多因子模型评估、政策周期配置和 mature admission 报告引用任务。 |
| 1.3 | 2026-06-27 | codex | 增补 Stage 3 通过型策略生产任务：候选搜索、通过门禁和 blocked 候选复盘。 |
| 1.4 | 2026-06-27 | codex | 增补异象发现 / 研究任务：候选登记、研究报告、Harvey / 单调性 / alpha / 时间切分 / 成本 / 经济逻辑准入。 |
| 1.5 | 2026-06-27 | codex | 增补自动异象发现系统任务：受控模板、批量 discovery、多重检验、动态目录接入和 Stage 3 候选消费。 |
| 1.6 | 2026-06-28 | codex | 增补研究引擎稳定模块整改任务：领域名模块、共享 helper、旧 engine 入口归档、脚本归档和测试引用迁移。 |

| Task ID | 任务 | 输入 | 输出 | 文件范围 | 验证 |
|---|---|---|---|---|---|
| FEAT-03-T01 | 维护 FactorSpec / FactorRunSpec 合同 | CR-030 HLD / ADR | schema 和错误码 | `engine/multifactor_contracts.py` | CR030 contract tests |
| FEAT-03-T01A | 维护统一因子目录 | 核心 7 因子、mature runner 5 因子、anomaly proxy 3 因子 | FactorCatalogEntry / status / usage / evidence refs | `engine/factor_registry.py` | `tests/test_factor_registry.py` |
| FEAT-03-T01B | 维护因子查询 CLI | factor registry | table / json 查询入口 | `scripts/list_factors.py` | `tests/test_list_factors_cli.py` |
| FEAT-03-T01B1 | 维护异象候选登记 | anomaly proxy、理论 / 行为 / 微观结构 / 财务扩展假设 | AnomalyCandidate、prior_logic_ref、economic_rationale、A 股调整说明 | `engine/anomaly_research.py` | `tests/test_chapter5_anomalies.py` |
| FEAT-03-T01B2 | 维护异象研究报告 | anomaly panel、anomaly returns、factor model returns、cost policy、split policy | AnomalyResearchReport、AnomalyAdmissionDecision、gap / risk warning | `engine/anomaly_research.py`、`scripts/research/run_anomaly_research.py` | `tests/test_chapter5_anomalies.py` |
| FEAT-03-T01B3 | 自动异象候选生成 | 受控字段模板、特征面板字段可用性 | ControlledAnomalyTemplate、AnomalyDefinition candidates、candidate matrices | `engine/anomaly_candidate_generator.py` | `tests/test_anomaly_candidate_generator.py` |
| FEAT-03-T01B4 | 自动异象批量研究 | feature panel、model returns、anomaly research gates | AnomalyDiscoveryRun、research report、admission decisions、markdown / parquet artifacts | `engine/anomaly_discovery.py`、`scripts/research/run_anomaly_discovery.py` | `tests/test_anomaly_discovery.py` |
| FEAT-03-T01B5 | 多重检验与动态目录接入 | anomaly research reports、admission decisions | Bonferroni / BH-FDR、dynamic FactorCatalogEntry extras、Stage 3 candidate FactorSpec | `engine/anomaly_multiple_testing.py`、`engine/anomaly_admission.py`、`engine/factor_registry.py`、`engine/mature_multifactor_research.py` | `tests/test_anomaly_discovery.py`、`tests/test_factor_registry.py`、`tests/test_stage3_mature_multifactor_research.py` |
| FEAT-03-T01C | Mature runner 因子目录纳管 | runner-local 因子 | 从目录构建 mature runner FactorSpec；旧 `stage3_*` 模块名已归档 | `engine/mature_multifactor_research.py`、`docs/legacy/archive/engine/stage3_mature_multifactor_research.py` | `tests/test_stage3_mature_multifactor_research.py` |
| FEAT-03-T01D | 维护高级多因子模型评估报告 | factor panel、labels、factor returns、portfolio、universe、tradability、policy cycle config | `factor_model_validation_report_v1`、gate decisions、risk warnings、blocked reasons、no-real-op counters | `engine/factor_model_validation.py`、`config/policy_cycles.yaml` | `tests/test_factor_model_validation_report.py`、`tests/test_policy_cycle_dataset_contract.py` |
| FEAT-03-T01E | Mature admission 消费评估报告 | mature runner valid panel、turnover、cost、label horizon、artifact refs | `FACTOR-MODEL-VALIDATION-REPORT.json`、`factor_model_validation_report_ref`、gate summary | `engine/mature_multifactor_research.py`、`engine/mature_multifactor_framework.py` | `tests/test_stage3_factor_model_validation_integration.py`、`tests/test_stage2_mature_multifactor_framework.py` |
| FEAT-03-T01F | 生产 Stage 3 通过型候选策略 | 当前 blocked run、数据湖 release、因子目录、评估报告、candidate configs | 通过型 mature multifactor strategy、PASS admission package、候选复盘和下一轮参数记录 | `engine/mature_multifactor_research.py`、后续 candidate sweep 入口 | `tests/test_stage3_factor_model_validation_integration.py`、后续 `tests/test_stage3_candidate_pass_gate.py` |
| FEAT-03-T01G | 候选搜索 / sweep 管理 | factor subset、weight scheme、rebalance、top_n、cost、risk constraints、train/test split | 每个候选的 validation report、blocked reason、pass candidate shortlist | 后续 `engine/mature_multifactor_candidate_search.py` 或等价领域名入口 | 后续 `tests/test_stage3_candidate_sweep.py` |
| FEAT-03-T01H | 研究引擎稳定模块整改 | 旧 chapter/stage/root 入口、重复 helper、测试引用 | 领域名模块、`docs/legacy/archive/engine/` 归档、`serialization`、`factor_research_matrices`、`admission_contracts`、稳定脚本入口 | `engine/factor_replication.py`、`engine/research_data_readiness.py`、`engine/factor_model_research.py`、`engine/anomaly_research.py`、`engine/factor_robustness.py`、`engine/factor_portfolio_practice.py`、`docs/legacy/archive/engine/*`、`scripts/research/*`、`scripts/qmt/*` | `tests/test_script_entrypoint_naming.py`、相关 research tests |
| FEAT-03-T02 | 维护 FactorPanel / LabelWindow gates | CR-011 + CR-030 | fail-closed panel / label policy | `engine/factor_panel_contracts.py` | label window tests |
| FEAT-03-T03 | 维护评价报告与组合器 | factor panel / benchmark / cost | report 和 portfolio plan | `engine/factor_evaluation.py`、`engine/multifactor_combiner.py` | evaluation / combiner tests |
| FEAT-03-T04 | 维护 ExperimentManifest / ReportCatalog | run spec / report path | 可复跑 manifest | `engine/research_manifest.py` | manifest catalog tests |
| FEAT-03-T05 | 维护 StrategyAdmissionPackage | reports / blocked claims | admission package / handoff | `engine/strategy_admission_package.py` | admission package tests |

## 后续触发条件

- 新增因子类别、标签窗口或组合器。
- 新增 factor_id、mature runner 因子、查询 CLI 参数或因子目录 schema。
- 新增或重命名 engine 研究主入口、旧 engine 入口归档、共享 serialization / matrix / admission contract 或脚本归档路径。
- 新增异象候选来源、Harvey 阈值、单调性规则、控制因子模型、成本 / 换手阈值、A 股特殊处理或经济逻辑准入规则。
- 新增或调整 FactorModelValidationReport schema、核心 gate policy、政策周期配置、GRS / 样本外 / 做空 / 壳价值 / 经济显著性评估规则。
- Stage 3 目标、pass gate、candidate sweep、候选参数空间或 blocked 候选复盘格式变化。
- 将 Qlib / external runner 从 reference 升级为 Spike。
- admission package 要进入 CR-021 simulation 申请输入。
