# CP3 CR091 QMT Strategy Runner HLD Review

## 自动预检摘要

自动预检结论：`PASS_WITH_RISK`  
auto_final_authorization: false  
不授权项：本 checkpoint 不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、submit / cancel、simulation / live。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP2 scope 已具备发起条件 | PASS | `process/checks/CP2-CR091-QMT-STRATEGY-RUNNER-SCOPE-CHECK.md`。 |
| HLD 已完成 | PASS | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md`。 |
| 研究依据已记录 | PASS | `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md`。 |
| 方案未依赖 runtime | PASS | 本轮只静态阅读本地代码和外部项目页面。 |

## Checklist

| 检查项 | 状态 | 说明 |
|---|---|---|
| 问题定义清楚 | PASS | HLD 第 1 节定位 runner gap。 |
| 成功标准量化 | PASS | HLD 第 2 节定义 6 条可度量标准。 |
| 候选方案对比充分 | PASS | HLD 第 3 节覆盖 5 个方案。 |
| 推荐方案明确 | PASS | 推荐 clean-room package-driven runner。 |
| 集成契约显式 | PASS | HLD 第 6 节覆盖调用方向、时机、输入、输出和降级。 |
| 相邻对象边界清楚 | PASS | HLD 第 7 节区分 CR046 / CR047 / CR089 / CR020 / CR092。 |
| 前置校验和失败路径完整 | PASS | HLD 第 8 节定义 fail closed 行为。 |
| 回退决策可操作 | PASS | HLD 第 9 节给出触发与回退目标。 |

## Decision Brief

推荐决策：批准 HLD 的 clean-room package-driven runner 架构。该架构的关键判断是：

- 多因子 admission package 是首选输入，但不写死成多因子专用 runner。
- `StrategyAdapter` 统一多因子、legacy strategy 和未来策略包。
- runner 统一输出 `TargetPortfolioSnapshot` 和 `OrderIntentDraftV1`。
- QMT 边界只保留 health / capabilities / query_positions，且默认 fake transport。
- 真实 runtime smoke 不属于 HLD approve 的授权内容。

备选方案：

| 备选 | 不推荐原因 |
|---|---|
| 外部执行器改造 | 外部项目多以真实账户、下单、自动启动或完整交易台为目标，首版安全裁剪成本高。 |
| 多因子专用 runner | 短期能跑多因子，但无法支持用户要求的其他策略运行能力。 |
| 只保留手工脚本 | 不能形成可复用验证边界，不能系统解决 runner gap。 |

### Context Capsule Summary

capsule: `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`、`process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`、`process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`  
read_profile: compact  
默认读取策略: capsule-first，优先读取上述 context capsule 和 checkpoint 摘要。  
全文档读取: 仅当 capsule 字段不足、人工审计或用户要求时读取完整 HLD / LLD / Test Plan。  

| Capsule | 摘要 |
|---|---|
| `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | CR091 scope 限定为静态研究、方案和人工门禁。 |
| `process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | HLD 推荐 clean-room package-driven runner，多因子优先且支持通用 StrategyAdapter。 |
| `process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | 后续实现仅限离线 runner 薄模块、checker、fixtures/tests 和 fake transport。 |

### Decision Collection Coverage

扫描状态: complete  
分类 / N/A 原因: 8 个候选问题均分类为 decision-item；无 N/A 项。  

| 来源 | 扫描结果 |
|---|---|
| HLD 成功标准 | 6 条均进入架构和测试约束。 |
| 本地代码静态证据 | 多因子、order intent、legacy strategy、QMT client / gateway contract 均纳入设计。 |
| 外部参考 | lite-qmt-executor、qmt-gateway、xqshare、vnpy_qmt 均作为参考，不作为首版依赖。 |
| 权限边界 | runtime、NAS、凭据、账户、下单和 simulation/live 均纳入不授权项。 |

候选问题数：8  
纳入待决策数：8  
N/A / 缺失原因：无。

### 待人工决策清单

用户需决策事项：本轮待人工决策项：8。本 CP3 重点确认 DQ-CP3-CR091-02、DQ-CP3-CR091-03、DQ-CP5-CR091-06。

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR091-01 | scope | CR091 当前是否启动为静态研究 / 方案 / 实施计划门禁？ | 启动 CR091 门禁，但 CP5 前不实现、不运行。 | 暂停；直接 CR047；直接实现。 | 推荐方案解决 runner 结构缺口且保留权限边界；直接实现会绕过设计门禁。 | 不证明真实 runtime 可用。 | 用户要求先交付策略包时切 CR047。 |
| DQ-CP3-CR091-02 | architecture | runner 主体路线是什么？ | clean-room package-driven runner。 | 改造 lite-qmt-executor；接入 qmt-gateway/xqshare；接入 vn.py；仅手工脚本。 | 推荐方案依赖少、边界清晰；外部项目实盘接口面过大。 | 需要自行实现 package/cache/evidence。 | 未来复杂事件驱动需求出现时再评估 vn.py / Lean。 |
| DQ-CP3-CR091-03 | architecture | 多因子与其他策略如何兼容？ | 多因子优先，统一 StrategyAdapter 输出合同。 | 多因子专用 runner；每类策略各自 runner。 | 推荐方案同时满足多因子优先和通用扩展；专用 runner 后续会分裂。 | adapter contract 需要稳定测试。 | 若多因子字段不稳定，先降级 legacy adapter。 |
| DQ-CP5-CR091-04 | implementation | 首版产物形态是什么？ | `trading/strategy_runner` 薄模块 + 离线 checker + fixtures/tests。 | 只写脚本；直接完整 runtime。 | 推荐方案可复用且仍小范围；脚本方案扩展弱，runtime 方案越权。 | 实现量高于单脚本。 | 若范围过大，可先交付 adapters + evidence 子集。 |
| DQ-CP5-CR091-05 | security | runner 是否直接从 NAS 运行策略？ | 不直接运行；只读本地 immutable cache / active pointer。 | NAS 原地运行；自动 pull/publish。 | 推荐方案避免 NAS 和发布边界；备选需新 CR。 | 后续需要单独处理包分发。 | 用户要求 NAS 自动取包时启动 CR093。 |
| DQ-CP5-CR091-06 | runtime_authorization | 首版是否允许真实 QMT runtime？ | 自动验证只允许 fake transport；真实只读 smoke 需后续逐 run 授权。 | 立即连接 QMT；完全取消只读 smoke。 | 推荐方案保留验证路线但不越权。 | 无真实 runtime ready 结论。 | 用户给出逐 run 授权后再开 runtime gate。 |
| DQ-CP5-CR091-07 | follow_up_tracking | 下单 / 撤单是否纳入 CR091？ | 不纳入，拆为 CR092 候选。 | 纳入 CR091 后半段。 | 推荐方案隔离 order write 风险；纳入会扩大门禁复杂度。 | CR091 不能验证下单链路。 | 用户明确要求订单写验证时启动 CR092。 |
| DQ-CP5-CR091-08 | risk_acceptance | 是否接受 CR091 首版不证明真实可交易？ | 接受；只证明 runner 合同和离线验证。 | 要求真实交易机验证后才通过；取消 runner。 | 推荐方案符合当前权限；真实验证需要额外授权。 | CP8 仍可能是 READY_WITH_RISK。 | 用户拒绝风险时暂停或新增 runtime gate。 |

## Exit Criteria

| 条目 | 状态 |
|---|---|
| HLD 可人工确认 | PASS |
| HLD 不扩大 runtime 授权 | PASS |
| HLD 可交接给 CP5 LLD / Test Plan | PASS_WITH_RISK |

## Deliverables

- `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md`
- `process/checks/CP3-CR091-QMT-STRATEGY-RUNNER-HLD-CHECK.md`
- `process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`

## 人工审查结果

状态：`approved`

审查人：user

审查时间：`2026-06-18T14:16:02+08:00`

用户输入：`同意`

结论：用户接受 DQ-CP3-CR091-02 和 DQ-CP3-CR091-03 推荐方案，同意采用 clean-room package-driven runner，并采用多因子优先、统一 `StrategyAdapter` 兼容其他策略的架构。该批准不授权真实 QMT / MiniQMT / XtQuant / gateway / runner runtime，也不授权交易写接口。
