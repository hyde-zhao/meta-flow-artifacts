# CP5 CR091 QMT Strategy Runner Readiness

## 自动预检摘要

自动预检结论：`PASS_WITH_RISK`  
auto_final_authorization: false  
不授权项：本 checkpoint 不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、submit / cancel、simulation / live。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| HLD 已具备确认条件 | PASS | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md` 与 CP3 check。 |
| LLD 草案已完成 | PASS | `process/stories/CR091-QMT-STRATEGY-RUNNER-LLD.md`。 |
| 测试计划已完成 | PASS | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md`。 |
| 实施尚未开始 | PASS | 本轮未新增 `trading/strategy_runner/` 代码，未运行未来测试命令。 |

## Checklist

| 检查项 | 状态 | 说明 |
|---|---|---|
| 文件影响范围明确 | PASS | LLD 列出模块、checker、tests、fixtures。 |
| Adapter 接口明确 | PASS | LLD 定义 `StrategyAdapter`、`AdapterResult`、`TargetPortfolioSnapshot`。 |
| 多因子优先 | PASS | `MultifactorAdmissionAdapter` 是首个主路径。 |
| 其他策略支持 | PASS | `LegacyStrategyResultAdapter` 和 `StrategyPackageAdapter` 已列入。 |
| 测试覆盖完整 | PASS | Test Plan 覆盖 static、contract、package、multifactor、legacy、fake gateway、redaction。 |
| 风险边界可审计 | PASS | forbidden counters 和 redaction assurance 已定义。 |
| CP5 approve 不等于 runtime 授权 | PASS | runtime 仍需后续逐 run authorization。 |

## Decision Brief

推荐决策：批准 CP5 readiness，允许后续进入离线 implementation slice：

- 新增 `trading/strategy_runner/` 薄模块。
- 新增离线 package checker。
- 新增脱敏 fixtures 和单元测试。
- 自动验证只使用 fake transport。
- 不启动 QMT / MiniQMT / XtQuant / gateway / runner runtime。
- 不访问 NAS、`.env`、凭据、账户、资金、持仓、委托、成交或日志原文。

备选方案：

| 备选 | 影响 |
|---|---|
| 只批准 HLD，不批准 CP5 | 可以继续讨论方案，但不进入实现。 |
| 只写脚本，不建模块 | 初始更小，但不利于多策略 adapter 扩展。 |
| 要求真实交易机 smoke 后再实现 | 当前权限不允许，且会把 runtime 授权前置。 |

### Context Capsule Summary

capsule: `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`、`process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`、`process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`  
read_profile: compact  
默认读取策略: capsule-first，优先读取上述 context capsule 和 checkpoint 摘要。  
全文档读取: 仅当 capsule 字段不足、人工审计或用户要求时读取完整 HLD / LLD / Test Plan。  

| Capsule | 摘要 |
|---|---|
| `process/context/CP2-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | CR091 scope 限定为静态研究、方案和人工门禁。 |
| `process/context/CP3-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | HLD 推荐 clean-room package-driven runner，多因子优先且支持通用 StrategyAdapter。 |
| `process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml` | CP5 approve 后仅允许离线 runner 薄模块、checker、fixtures/tests 和 fake transport。 |

### Decision Collection Coverage

扫描状态: complete  
分类 / N/A 原因: 8 个候选问题均分类为 decision-item；无 N/A 项。  

| 来源 | 扫描结果 |
|---|---|
| HLD | 已抽取架构、边界、ADR 候选、Gotchas。 |
| LLD | 已抽取文件影响范围、接口草案、adapter 映射、实施顺序、回滚策略。 |
| Test Plan | 已抽取测试层级、fixtures、禁止操作断言和未来命令。 |
| CR091 原计划 | 已继承原 DQ-CP2-CR091-01..06，并按多因子优先和 module-first 方案补强。 |

候选问题数：8  
纳入待决策数：8  
N/A / 缺失原因：无。

### 待人工决策清单

用户需决策事项：本轮待人工决策项：8。

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR091-01 | scope | CR091 当前是否启动为静态研究 / 方案 / 实施计划门禁？ | 启动 CR091 门禁，但 CP5 前不实现、不运行。 | 暂停；直接 CR047；直接实现。 | 推荐能先解决 runner 结构缺口；直接实现会绕过设计门禁。 | 不证明真实 runtime 可用。 | 用户要求先交付策略包时切 CR047。 |
| DQ-CP3-CR091-02 | architecture | runner 主体路线是什么？ | clean-room package-driven runner。 | 改造 lite-qmt-executor；接入 qmt-gateway/xqshare；接入 vn.py；仅手工脚本。 | 推荐依赖少、边界清晰；外部项目实盘接口面过大。 | 需要自行实现 package/cache/evidence。 | 未来复杂事件驱动需求出现时再评估 vn.py / Lean。 |
| DQ-CP3-CR091-03 | architecture | 多因子与其他策略如何兼容？ | 多因子优先，统一 StrategyAdapter 输出合同。 | 多因子专用 runner；每类策略各自 runner。 | 推荐同时满足多因子优先和通用扩展；专用 runner 后续会分裂。 | adapter contract 需要稳定测试。 | 若多因子字段不稳定，先降级 legacy adapter。 |
| DQ-CP5-CR091-04 | implementation | 首版产物形态是什么？ | `trading/strategy_runner` 薄模块 + 离线 checker + fixtures/tests。 | 只写脚本；直接完整 runtime。 | 推荐可复用且仍小范围；脚本方案扩展弱，runtime 方案越权。 | 实现量高于单脚本。 | 若范围过大，可先交付 adapters + evidence 子集。 |
| DQ-CP5-CR091-05 | security | runner 是否直接从 NAS 运行策略？ | 不直接运行；只读本地 immutable cache / active pointer。 | NAS 原地运行；自动 pull/publish。 | 推荐避免 NAS 和发布边界；备选需新 CR。 | 后续需要单独处理包分发。 | 用户要求 NAS 自动取包时启动 CR093。 |
| DQ-CP5-CR091-06 | runtime_authorization | 首版是否允许真实 QMT runtime？ | 自动验证只允许 fake transport；真实只读 smoke 需后续逐 run 授权。 | 立即连接 QMT；完全取消只读 smoke。 | 推荐保留验证路线但不越权。 | 无真实 runtime ready 结论。 | 用户给出逐 run 授权后再开 runtime gate。 |
| DQ-CP5-CR091-07 | follow_up_tracking | 下单 / 撤单是否纳入 CR091？ | 不纳入，拆为 CR092 候选。 | 纳入 CR091 后半段。 | 推荐隔离 order write 风险；纳入会扩大门禁复杂度。 | CR091 不能验证下单链路。 | 用户明确要求订单写验证时启动 CR092。 |
| DQ-CP5-CR091-08 | risk_acceptance | 是否接受 CR091 首版不证明真实可交易？ | 接受；只证明 runner 合同和离线验证。 | 要求真实交易机验证后才通过；取消 runner。 | 推荐符合当前权限；真实验证需要额外授权。 | CP8 仍可能是 READY_WITH_RISK。 | 用户拒绝风险时暂停或新增 runtime gate。 |

## 不授权项

如果用户回复 `approve`，只表示接受上表 8 项推荐方案，不表示授权以下动作：

- 不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime 启动。
- 不授权 NAS read / write / list / copy / pull / publish。
- 不授权读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- 不授权 submit / cancel。
- 不授权 simulation / live。
- 不授权 provider / lake / catalog / publish。
- 不授权恢复 CR020 的用户删除路线。
- 不授权自动启动 CR089 runtime。

## Exit Criteria

| 条目 | 状态 |
|---|---|
| CP5 readiness 可人工确认 | PASS_WITH_RISK |
| 允许后续离线实现范围明确 | PASS |
| runtime 仍被阻断 | PASS |

## Deliverables

- `process/checks/CP5-CR091-QMT-STRATEGY-RUNNER-READINESS-CHECK.md`
- `process/context/CP5-CR091-QMT-STRATEGY-RUNNER-CONTEXT.yaml`
- `process/stories/CR091-QMT-STRATEGY-RUNNER-LLD.md`
- `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md`

## 人工审查结果

状态：`approved`

审查人：user

审查时间：`2026-06-18T14:16:02+08:00`

用户输入：`同意`

结论：用户接受 DQ-CP5-CR091-04..08 推荐方案，同意进入 CR091 离线实现切片：`trading/strategy_runner` 薄模块、离线 checker、fixtures/tests、fake transport 和脱敏 evidence。批准范围仍不包含 NAS 原地运行 / 自动 pull / publish、真实 QMT runtime、`.env` / 凭据 / 账户读取、submit / cancel、simulation / live、provider / lake / publish。下单 / 撤单拆为 CR092 候选，NAS package 分发拆为 CR093 候选。
