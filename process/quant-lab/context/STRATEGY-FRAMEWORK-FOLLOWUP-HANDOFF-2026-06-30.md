# Strategy Framework Follow-up Handoff

## 目的

本交接文档把当前 experiment 整改中需要落入后续需求的内容，分配到多因子策略、机器学习策略和事件驱动策略三个后续框架。当前文档不启动这些框架，不授权真实 lake、runtime 或交易。

## 总路线

1. Experiment 离线整改全清。
2. E4 真实 lake readonly semantic validation。
3. 多因子策略全流程生产框架。
4. 机器学习策略全流程框架。
5. 事件驱动策略全流程框架。

## 多因子策略框架输入

| 当前整改产物 | 应落入后续需求 | 说明 |
|---|---|---|
| `engine.factor_statistics` 的 forward return、sort、IC/Rank IC | 多因子研究标准统计合同 | M2 多因子研究标准化应复用这些合同 |
| `engine.factor_library` core factor set | 多因子长期因子身份库 | M2/M2.5 必须区分 core、experiment-local、research candidate |
| `factor_ownership_matrix` | 因子准入前置 | 生产 core 需要 calculator 可用和真实数据语义验证 |
| `research_only_admission_summary` 边界 | M2.5 研究产物准入契约 | 研究摘要不能自动变成生产 admission package |
| CR140/142 synthetic parity 边界 | E4/M1 真实数据验证 | 真实 lake 只读验证通过前不得生产化 |

## 多因子生产阶段建议

| 阶段 | 目标 |
|---|---|
| M1 | 数据接入与真实 lake 只读验证 |
| M2 | 多因子研究标准化 |
| M2.5 | 研究产物准入契约：research result 到 `StrategyAdmissionPackage` |
| M3 | 生产级回测 |
| M4 | Strategy admission / risk gate |
| M5 | Paper simulation |
| M6 | Live readiness |
| M7 | Live / QMT integration |

## 机器学习策略框架输入

| 当前整改产物 | 应落入后续需求 | 说明 |
|---|---|---|
| `run_experiment_23_29_ml_factor_suite.py` 回归通过 | ML 离线研究入口 | 当前只证明离线研究 smoke，不证明生产 |
| factor ownership 规则 | ML feature / factor 归属 | ML 特征不能绕过 factor ownership 和数据语义验证 |
| E4 只读验证结果 | ML 训练数据语义前置 | 真实数据语义未验证前，不做 model registry / serving |
| CR139 training snapshot contract 线索 | ML training snapshot lineage | 后续 ML 框架需要完整训练快照、模型版本和回滚 |

## 机器学习后续必须补的生产合同

- Model registry。
- Model artifact versioning。
- Training snapshot provenance。
- Offline / online feature parity。
- Prediction logging。
- Drift monitoring。
- Retraining governance。
- Model rollback。
- Admission / paper simulation / live consumer contract。

## 事件驱动策略框架输入

| 当前整改产物 | 应落入后续需求 | 说明 |
|---|---|---|
| `engine.events`、data lake events schema 相关测试 | 事件数据基础 | 需在 E4 后确认事件字段真实语义 |
| factor/statistics 合同 | 事件策略研究评估 | 事件驱动策略仍需要统一收益标签、窗口、统计口径 |
| strategy admission / paper simulation 既有模块 | 事件策略生产准入 | 后续复用多因子生产框架的 admission 和 paper simulation |

## 事件驱动策略后续必须补的生产合同

- 事件定义、来源、去重、发生时间和可得时间。
- 事件窗口和标签生成规则。
- 事件冲击研究、组合构建和持有期规则。
- 事件策略 admission package。
- 事件策略 paper simulation。
- 事件策略 live readiness、风控、监控和回滚。

## 当前不能声明的能力

- 不能声明多因子策略已完成全流程生产。
- 不能声明机器学习策略已完成生产化。
- 不能声明事件驱动策略已完成生产化。
- 不能声明真实 lake 语义已验证。
- 不能声明模拟盘或实盘已接入。
