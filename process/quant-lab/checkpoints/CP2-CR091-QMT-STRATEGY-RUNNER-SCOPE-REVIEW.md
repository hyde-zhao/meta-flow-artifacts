# CP2 CR091 QMT Strategy Runner Scope Review

## 自动预检摘要

自动预检结论：`PASS_WITH_RISK`  
auto_final_authorization: false  
不授权项：本 checkpoint 不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、submit / cancel、simulation / live。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| 用户明确启动 CR091 | PASS | 用户选择“启动 CR091”，并补充要求“优先考虑多因子策略，同时具备其他策略的运行能力”。 |
| CR046 已关闭 | PASS | CR046 于 2026-06-18T07:59:20+08:00 关闭为 `closed-current-delivery / READY_WITH_RISK`。 |
| 当前无其他 active formal CR | PASS | CR046 关闭后 active_change 清空；本轮仅启动 CR091 门禁。 |
| 权限边界已确认 | PASS | 不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env`、账户、submit / cancel、simulation / live。 |

## Checklist

| 检查项 | 状态 | 说明 |
|---|---|---|
| CR091 scope 是否是 runner research / design / implementation plan | PASS | 本轮只生成研究、HLD、LLD、测试计划和门禁材料。 |
| 是否解决“runner 没开发出来如何验证策略”的结构缺口 | PASS | 方案引入 StrategyAdapter、target portfolio、order intent draft、fake readonly gateway 和 redacted evidence。 |
| 是否多因子优先 | PASS | 多因子 admission package 被设为首选输入。 |
| 是否支持其他策略 | PASS | 通过 `LegacyStrategyResultAdapter` 和 `StrategyPackageAdapter` 支持 RSI / MACD / momentum 与后续策略包。 |
| 是否避免扩大到交易写接口 | PASS | submit / cancel / simulation / live 明确排除。 |

## Decision Brief

推荐决策：批准 CR091 进入静态研究 / HLD / LLD / 测试计划确认，并采用 clean-room package-driven runner 方案。批准后仅允许后续离线实现和 fixture / fake transport 测试，不授权任何 runtime。

备选方案：

| 备选 | 影响 |
|---|---|
| 直接实现 QMT runner | 速度快但权限边界不清，会把 runner gap 与真实交易风险混在一起。 |
| 改造外部 QMT 执行器 | 可复用部分调度经验，但账户、密码、下单和自动启动风险过高。 |
| 暂停 CR091，先做 CR047 | 可先交付具体策略包，但 runner 缺口仍存在。 |

### Context Capsule Summary

capsule: `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`、`process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`、`process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`  
read_profile: compact  
默认读取策略: capsule-first，优先读取上述 context capsule 和 checkpoint 摘要。  
全文档读取: 仅当 capsule 字段不足、人工审计或用户要求时读取完整 HLD / LLD / Test Plan。  

| Capsule | 摘要 |
|---|---|
| `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | CR091 scope 限定为静态研究、方案和人工门禁；不授权 runtime、NAS、凭据、账户或交易动作。 |
| `process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | HLD 推荐 clean-room package-driven runner，多因子优先且支持通用 StrategyAdapter。 |
| `process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | 后续实现仅限 `trading/strategy_runner` 薄模块、离线 checker、fixtures/tests 和 fake transport。 |

### Decision Collection Coverage

扫描状态: complete  
分类 / N/A 原因: 8 个候选问题均分类为 decision-item；无 N/A 项。  

| 来源 | 扫描结果 |
|---|---|
| `process/STATE.md` | 已扫描 CR046 关闭、CR089 / CR091 blocked 事实和不授权项。 |
| `process/changes/CR-091-...md` | 已扫描旧 DQ-CP2-CR091-01..06，并按本轮启动事实更新。 |
| `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md` | 已纳入外部参考与本地基线结论。 |
| `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md` | 已纳入推荐架构与备选方案。 |
| `process/stories/CR091-QMT-STRATEGY-RUNNER-LLD.md` | 已纳入实施范围与文件边界。 |

候选问题数：8  
纳入待决策数：8  
N/A / 缺失原因：无。所有高风险范围、架构、安全、实施、运行授权和后续跟踪问题均进入人工决策队列。

### 待人工决策清单

用户需决策事项：本轮待人工决策项：8。

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
| Scope decision brief 完成 | PASS |
| 待人工决策项已列出 | PASS |
| 不授权项已独立列出 | PASS |
| 可发起人工确认 | PASS_WITH_RISK |

## Deliverables

- `process/checks/CP2-CR091-QMT-STRATEGY-RUNNER-SCOPE-CHECK.md`
- `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`
- `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md`

## 人工审查结果

状态：`approved`

审查人：user

审查时间：`2026-06-18T14:16:02+08:00`

用户输入：`同意`

结论：用户接受 DQ-CP2-CR091-01 推荐方案，同意 CR091 进入静态研究 / 方案 / 实施计划门禁。该批准不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env` / 凭据 / 账户、submit / cancel、simulation / live、provider / lake / publish。
