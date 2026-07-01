---
title: "Quant Research Production Roadmap"
status: "draft-v0.3"
owner: "host-orchestrator"
created_at: "2026-07-01"
scope: "data lake, research/backtest framework, paper/live operations"
authorization_boundary: "planning-only; no runtime, broker write, provider fetch, NAS sync, credential read, or live trading authorization"
---

# Quant Research Production Roadmap

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-07-01 | host-orchestrator | 初版路线图：按数据湖、研究生产、回测框架、paper/live、实盘操作划分阶段目标。 |
| v0.2 | 2026-07-01 | host-orchestrator | 根据 `process/checks/QUANT-RESEARCH-PRODUCTION-ROADMAP-REVIEW-2026-07-01.md` 补 NAS 多节点一致性、既有资产衔接、量化 exit、fail-closed / pointer 门、日频边界和 Gotchas。 |
| v0.3 | 2026-07-01 | host-orchestrator | 回写 CR146-CR150 实施进度：本地数据湖 Phase 1 closed with NAS sync deferred，研究/回测 foundation 已闭环，下一 active CR 为 CR150 多因子框架补齐。 |

## 1. 目标

本路线图定义 quant-lab 后续从当前数据湖与研究脚本体系，演进到支持多因子策略、机器学习策略和事件驱动策略的全流程研究生产、回测验证、paper trading、readonly live 和受控实盘的阶段目标。

本路线图以**日频多因子研究生产闭环**作为第一条 baseline 路线；分钟级、Level2、Qlib / Backtrader 深度集成和 live event trading 保持 deferred / Spike 边界。若后续恢复 CR019 deferred capabilities register，应把该 register 作为日内 / Level2 / 外部框架能力的边界来源；在 register 未恢复前，相关能力不得进入 CR-147 默认范围。

总体目标不是先打通下单，而是先保证：

1. 数据事实 PIT-safe、可审计、可复现。
2. 研究样本、特征、标签、事件和模型版本可追溯。
3. 回测结果可比较、可复跑、可晋级。
4. paper / readonly live 与离线研究口径一致。
5. 真实下单独立授权，且受 OMS、风控、审计和 kill switch 控制。

## 2. 当前基线

基于 CR139 / CR146 evidence，当前数据湖状态可概括为：

| 能力 | 当前状态 | 生产级差距 |
|---|---|---|
| Catalog current truth | 17/17 datasets published | 需要持续 publish、rollback、drift validation 机制 |
| Physical existence | 17/17 physical path exists | N1 physical partition / archive lifecycle 未完成 |
| Lineage checksum | 17/17 present | 需要保证未来写入也强制 checksum lineage |
| Reader runtime | PIT / panel / read audit smoke PASS | 需要 recurring validation 和 SLA |
| Duplicate profile | 已真实扫描 | 需要 dataset-level conflict policy 和 quarantine/write protocol |
| Business conflict | 已量化 4,272,624 groups | 需要语义规则或 full-group quarantine 决策 |
| PIT readiness | 部分 dataset 显式 `pit_available` | 需要 17 dataset 全量 readiness matrix |
| Run ordering | 有 deterministic fallback | 需要 N2 run-id governance / run registry |
| Live / broker operation | 未授权 | 需要 readonly live、paper、small-live 分阶段授权 |

当前可接受定位：

> 当前数据湖支持 research current truth 与读层验证，但还不是完整 production-grade governed lake。后续应先补数据事实治理，再推进研究生产化和实盘。

### 2.1 当前实施进度（2026-07-01）

| 范围 | CR / Evidence | 当前状态 | 说明 |
|---|---|---|---|
| CR139 post-close lake completion | `CR-146` | closed / READY | 已完成真实 lake runtime validation、fail-closed reader 加固、N1 current-truth 物理分区迁移、catalog pointer 切换和历史 cleanup 分流。 |
| Phase 2 research production foundation | `CR-147` | closed / READY | 已完成 ResearchDatasetSpec、feature/label/training snapshot/experiment/admission 资产收敛；asset map gap=0。 |
| Phase 3 backtest foundation | `CR-148` | closed / READY | 已完成 BacktestRunSpec、report pack、cost/risk/attribution metadata contract；相关本地测试通过。 |
| Phase 1 governed lake readiness | `CR-149` | closed-current-delivery / READY_WITH_RISK | 本地 17-dataset readiness、PIT、run registry、business-conflict quarantine policy 和 recurring validation plan 已闭环。NAS current-truth sync 因用户决策后置。 |
| NAS current-truth scoped sync | `RA-CR149-001` | candidate / deferred | 已有只读 consistency evidence：NAS catalog stale，17 个 NAS `current/` 路径缺失。后续恢复时需单独 approve scoped local-to-NAS sync。 |
| Multifactor framework completion | `CR-150` | active / no-risk implementation | 当前下一阶段：补齐日频多因子 factor spec → signal set → portfolio construction → backtest spec → report/admission linkage。 |

当前执行锁：

```text
active formal CR: CR-150 Multifactor Framework Completion
deferred runtime candidate: RA-CR149-001 NAS current-truth scoped sync
```

执行边界：

- CR150 可以链接数据湖 metadata contract / catalog refs / readiness / PIT refs。
- CR150 当前不读取或写入真实 lake，不同步 NAS，不读取 provider，不启动 simulation/live/trading，不写 broker。
- 若后续需要真实 lake factor panel / label 构建、Feature Store 写入、NAS report 写入或 runtime execution，必须另起人工门禁。

## 3. 能力蓝图

### 3.1 分层架构

```text
Data Foundation
  - production data lake
  - PIT contract
  - feature store
  - label store
  - event store
  - run registry

Research Production
  - ResearchDatasetSpec
  - factor research
  - ML training
  - event replay
  - experiment registry
  - model / strategy registry

Backtest and Simulation
  - strategy interface
  - portfolio simulator
  - cost / slippage model
  - tradability constraints
  - attribution
  - report pack

Live Operations
  - readonly live
  - paper OMS
  - broker adapter
  - OMS state machine
  - pre-trade risk
  - kill switch
  - reconciliation

Multi-node Operations
  - NAS-backed shared view
  - write-primary protocol
  - catalog / pointer consistency
  - disaster recovery drill
```

### 3.2 依赖方向

| 上游 | 下游 | 规则 |
|---|---|---|
| Data Lake | Research Dataset Builder | 下游只能通过 PIT-safe builder 消费数据，不直接拼 raw/canonical 文件 |
| Feature / Label / Event Store | Strategy Research | 策略研究依赖版本化特征、标签和事件，不自行定义数据事实 |
| Strategy Research | Backtest Engine | 策略输出 signal / target，不直接决定真实订单 |
| Backtest Engine | Promotion Gate | 只有通过验证的 strategy artifact 可进入 paper / live |
| Paper / Readonly Live | Small-live | 真实下单必须独立授权，不能由研究阶段继承 |
| OMS / Risk Engine | Broker Adapter | broker write 只能由 OMS 经风控后发起 |
| NAS Shared View | Research / Live Nodes | 多节点只能读一致 catalog / pointer；写入必须经 write-primary 或显式独占窗口 |

禁止依赖：

| 禁止方向 | 原因 |
|---|---|
| Strategy 直接写 broker order | 绕过 OMS、风控、审计和 kill switch |
| ML training 直接读非 PIT lake 文件 | 易引入 lookahead / leakage |
| Event strategy 使用 event_time 替代 available_at | 会高估事件策略表现 |
| Backtest 使用未版本化 feature / label | 无法复现和晋级 |
| Data cleanup 自动选择 business-conflict 主记录 | 会污染生产事实 |
| 多节点直接并发写 lake / catalog / pointer | 会破坏 current truth 一致性和 rollback 语义 |

### 3.3 既有资产衔接

后续 CR 不应从零重建已有能力，而应优先收敛、补契约、补门禁和补证据。

| 路线图对象 | 既有资产 | 后续定位 |
|---|---|---|
| Feature / Label Store | `market_data/features/artifacts.py` | 将 `FeatureArtifactSpec`、`LabelArtifactSpec`、`FeatureStoreSwitchPolicy` 升级为正式 Feature / Label Store contract |
| Experiment registry | `engine/research_manifest.py`, `engine/training_snapshot_contract.py` | 收敛为 experiment / training snapshot 机器真相源 |
| Strategy artifact / admission | `engine/strategy_admission_package.py`, `engine/strategy_readiness_admission.py` | 作为 strategy artifact 和 promotion gate 的起点 |
| Paper simulation | `engine/paper_simulation.py` | 作为 Phase 5 paper OMS / paper portfolio 的基础能力 |
| Backtest / multifactor framework | `engine/backtest.py`, `engine/mature_multifactor_framework.py` | 作为 Phase 3 baseline，不重建平行回测引擎 |
| Strategy runner / simulation gates | `trading/strategy_runner/` | 作为 Phase 5 / Phase 6 readonly、simulation、admission、kill-switch 相关能力的收敛对象 |
| Stable data-lake scripts | `scripts/data_lake/*.py` | 新自动化必须使用稳定入口，不使用 `scripts/legacy/cr/*` 作为执行面 |

## 4. 阶段目标

### Phase 0：CR139 / CR146 基线收敛

目标：把当前数据湖状态固化为后续工作的可信起点。

| 项 | 目标 |
|---|---|
| CR146 close | 经 CP2 approve 后关闭为 `READY_WITH_RISK` |
| Follow-up tracking | 保留 CR139 高风险项单一台账 |
| Evidence baseline | inventory、golden baseline、reader smoke、duplicate profile、split planning 全部可追溯 |
| Authorization boundary | provider / NAS / runtime / trading / credential / Git remote 不被隐式授权 |
| Fail-closed fallback hardening | 加固 reader rglob-all / source_run_id fallback，避免 catalog 回归时按字典序误选 survivor |

量化退出条件：

| 条件 | 门槛 |
|---|---|
| CR tracking | `meta-flow check cr-tracking` PASS，且 CR-146 为唯一 active formal CR |
| Human gate | `meta-flow check human-gate --checkpoint ... --launch-message-file ...` PASS |
| Evidence baseline | inventory、golden、reader smoke、duplicate profile、split planning 5 类 CR146 evidence JSON 全部有效 |
| Stable script surface | CR146 新证据命令 100% 使用 `scripts/data_lake/*` 稳定入口 |
| Follow-up tracking | CR139 A/B/C 类开放项 100% 进入 `CR-139-FOLLOW-UP-TRACKING` 或正式 CR |
| Fail-closed fallback | reader fallback 加固测试覆盖 catalog missing / unpublished / source_run_id ambiguity 3 类场景，且 0 新失败 |

### Phase 1：生产级数据湖治理

目标：把 current truth 可读的数据湖，升级为核心 dataset 有生产治理规则的数据湖。

优先治理对象：

| 优先级 | Dataset | 原因 |
|---:|---|---|
| 1 | `trade_status` | business-conflict groups 最大，影响可交易性、停牌、样本过滤 |
| 2 | `prices_limit` | 影响涨跌停、成交约束和实盘风控 |
| 3 | `prices` | 影响收益率、标签、模型训练和回测 |
| 4 | `events` | 影响事件驱动触发和 available_at 语义 |
| 5 | `adj_factor` | 无 business conflict，但需要 metadata-only precedence |

阶段目标：

| 能力 | 目标 |
|---|---|
| Dataset readiness matrix | 每个 dataset 明确 `production_ready / research_ready / quarantined / unsupported` |
| Conflict policy | exact-copy、metadata-only、business-conflict 分别有正式规则 |
| Quarantine zone | business-conflict 默认进入 quarantine，不自动择优 |
| PIT contract | 每个 dataset 明确 PIT key、available_at、decision_time、timezone |
| Run registry | 替代 source_run_id 字典序 fallback |
| Data quality gates | 每次写入都有 row count、checksum、schema diff、coverage diff |
| Recurring validation | inventory / golden / PIT smoke / duplicate scan 可定期运行 |
| Published pointer consistency | published pointer / catalog current pointer 明确为多节点一致视图来源 |
| Multi-node consistency | 定义 NAS shared-read、write-primary、exclusive write window、DR fallback 和 catalog/pointer sync gate |

不优先执行：

- N1 physical partition migration。
- legacy delete / archive。
- provider fetch / uncontrolled NAS sync。

Phase 1 中的 NAS 目标是**一致性协议和门禁设计**，不等于立即执行 NAS sync。真实 NAS 读写、同步或灾备演练仍需独立 runtime authorization。

量化退出条件：

| 条件 | 门槛 |
|---|---|
| Readiness matrix | 17/17 catalog datasets 有 `production_ready / research_ready / quarantined / unsupported` 之一 |
| P0 conflict policy | `trade_status`、`prices_limit`、`prices` 3/3 通过 dataset-level human gate 或进入 explicit quarantine policy |
| P1 conflict policy | `events`、`adj_factor` 2/2 有 exact-copy / metadata-only / business-conflict 处理策略 |
| Business-conflict handling | 4,272,624 business-conflict groups 100% 分类为 quarantine / semantic-rule-required / unsupported，不要求清零 |
| PIT readiness | 17/17 datasets 有 `pit_available / not_applicable / unsupported-with-reason`，不允许长期 `null` |
| Run registry | 100% 新写入 run 具有 registry record；reader 不再依赖 source_run_id 字典序作为生产排序 |
| Published pointer | 17/17 published datasets 的本地 current pointer 可读且 checksum / run_id 完整；多节点一致性可 deferred 到 `RA-CR149-001` |
| NAS consistency gate | 已完成 1 次 read-only consistency check；发现 NAS stale，scoped sync 后置，不阻塞本地 CR150 |

Phase 1 的目标是建立 quarantine policy 和 dataset-level 决策流程，不是把 4,272,624 个冲突全部语义择优或物理清零。2026-07-01 当前状态为：本地治理目标已完成；NAS shared-view sync 留作 `RA-CR149-001` 后续 runtime authorization，不作为 CR150 本地多因子框架补齐的前置阻断。

### Phase 2：研究数据集、Feature Store、Label Store、Event Store

目标：建立多因子、机器学习、事件驱动共用的 PIT-safe 样本构建体系。

核心对象：

```text
ResearchDatasetSpec
  - universe
  - start_date / end_date
  - as_of
  - features
  - labels
  - event_filters
  - tradability_filters
  - rebalance_calendar
  - leakage_policy
  - output_snapshot_id
```

阶段目标：

| 能力 | 多因子 | ML | 事件驱动 |
|---|---|---|---|
| Feature Store | factor value、标准化、中性化 | model features、feature version | event-derived features |
| Label Store | forward return、rank label | train labels、classification/regression target | event-window return |
| Event Store | 可选 | 可选 | event_time、available_at、decision_time |
| Snapshot | factor panel snapshot | train/valid/test snapshot | event replay snapshot |
| Leakage Gate | factor availability | feature/label split | event availability |

量化退出条件：

| 条件 | 门槛 |
|---|---|
| Spec replay | 同一 `ResearchDatasetSpec` 连续重跑 2 次，row count、primary key checksum、feature checksum 100% 一致 |
| PIT-safe consumption | 新增研究样本构建入口 100% 通过 PIT-safe builder，不直接读 raw / canonical parquet |
| Snapshot metadata | 100% 样本集包含 dataset snapshot、feature version、label version、universe version |
| Leakage gate | feature `available_at <= decision_time` 覆盖率 100%；label window overlap violation 为 0 |
| Existing asset convergence | `market_data/features/artifacts.py` 与 `engine/training_snapshot_contract.py` 被纳入正式 contract，而非重建平行实现 |
| Multi-node visibility | 1 个 ResearchDatasetSpec snapshot 在至少 2 个节点可读，snapshot checksum 一致 |

### Phase 3：统一回测与实验框架

目标：让多因子策略先用统一回测、成本、约束、归因和报告框架跑通 research -> backtest -> experiment registry 闭环；ML 和事件策略的 baseline 接入在 Phase 3.5 / Phase 4 之后推进。

统一策略接口：

```text
SignalStrategy
  -> input: features / events / market state
  -> output: alpha score / signal

PortfolioPolicy
  -> input: signal + constraints
  -> output: target weights / target positions

ExecutionPolicy
  -> input: target positions + tradability + market state
  -> output: simulated orders
```

阶段目标：

| 模块 | 目标 |
|---|---|
| Portfolio simulator | 持仓、现金、调仓、成交、费用、滑点 |
| Cost model | commission、tax、slippage、market impact |
| Tradability constraints | 停牌、涨跌停、成交量、黑名单、交易日历 |
| Risk model | 行业、市值、风格、集中度、换手、回撤 |
| Experiment registry | config、data snapshot、code version、metrics |
| Attribution | 收益、成本、风格、行业、因子贡献拆解 |
| Report pack | JSON / Markdown / HTML 固定格式 |

量化退出条件：

| 条件 | 门槛 |
|---|---|
| 多因子 baseline | 至少 1 个日频多因子 baseline 完成 full backtest，并生成 report pack |
| Experiment registry | 100% Phase 3 实验有 `experiment_id`、config hash、data snapshot、code version、metrics |
| Cost / slippage | baseline 回测 100% 纳入 commission、tax、slippage 至少 3 类成本字段 |
| Risk report | baseline report 包含行业、市值、换手、集中度、回撤 5 类风险指标 |
| Reproducibility | 同一 experiment rerun 2 次，核心 metrics 差异在 1e-9 或报告 checksum 一致 |
| ML / event boundary | ML / event baseline 不作为 Phase 3 退出阻断；进入 Phase 3.5 / Phase 4 前必须完成 Phase 2 store contract |

### Phase 4：策略生产化流水线

目标：把研究脚本升级为可晋级、可部署、可回滚的 strategy artifact。

Strategy artifact 建议结构：

```text
strategy.yaml
features.yaml
dataset_snapshot.json
backtest_report.json
risk_limits.yaml
promotion_record.json
rollback_plan.md

# ML 策略额外包含
model.pkl / model.onnx
training_config.yaml
prediction_schema.json
drift_report.json
```

Promotion gates：

| Gate | 检查目标 |
|---|---|
| Research Gate | 回测结果、样本量、统计稳定性 |
| Leakage Gate | PIT / available_at / label window 无泄漏 |
| Robustness Gate | 分年份、分行业、分市场状态稳定 |
| Cost Gate | 加成本后仍有效 |
| Capacity Gate | 容量、冲击成本、换手可接受 |
| Risk Gate | 暴露、集中度、回撤、黑名单 |
| Paper Gate | paper 结果与离线预期一致 |
| Ops Gate | 可监控、可暂停、可回滚 |

量化退出条件：

| 条件 | 门槛 |
|---|---|
| Artifact coverage | 100% 晋级候选策略包含 strategy.yaml、dataset_snapshot.json、backtest_report.json、risk_limits.yaml、promotion_record.json、rollback_plan.md |
| Promotion gate | 100% 候选策略得到 `pass / fail / needs_review` 之一，不允许空状态 |
| Existing asset convergence | `engine/strategy_admission_package.py` 和 `engine/strategy_readiness_admission.py` 被用作 admission 起点 |
| ML artifact | ML 候选若进入本阶段，100% 包含 model artifact、training_config、prediction_schema、drift_report |
| NAS / multi-node precheck | 进入 Phase 5 前，strategy artifact 在至少 2 个节点可读且 hash 一致 |

### Phase 5：Readonly Live 与 Paper Trading

目标：验证实时链路，不真实下单。

Readonly live：

| 能力 | 目标 |
|---|---|
| Market data readonly | 实时行情接入，只读 |
| Account / position readonly | 账户、资金、持仓只读 |
| Live feature builder | 实时特征与离线口径一致 |
| Live signal shadow | 生成 shadow signal，不下单 |
| Live-vs-offline reconciliation | 同日 replay 与 live signal 差异可解释 |

Paper trading：

| 能力 | 目标 |
|---|---|
| Paper OMS | 模拟订单生命周期 |
| Simulated fills | 成交模拟、滑点、费用 |
| Paper portfolio | paper position、cash、PnL |
| Paper risk | 使用真实 pre-trade risk 规则 |
| Daily report | 每日 paper report 和异常摘要 |

前置条件：

- Phase 1 NAS shared-read / pointer consistency gate PASS。
- Phase 2 snapshot 多节点可见性 PASS。
- 仍不授权真实 order write。

量化退出条件：

| 条件 | 门槛 |
|---|---|
| Readonly live duration | 连续 >=10 个交易日 |
| Paper duration | 连续 >=10 个交易日 |
| Live-vs-offline reconciliation | 每日 live signal vs offline replay 差异 100% 可解释；不可解释差异为 0 |
| Paper OMS audit | 100% paper orders 有 submitted / accepted / filled_or_cancelled / reconciled 状态链 |
| No real order write | broker write operation count = 0 |
| Multi-node consistency | 至少 2 个节点读取同一 strategy artifact、dataset snapshot、catalog pointer，hash 100% 一致 |

### Phase 6：受控小额实盘

目标：在强风控、强审计、低资金、白名单范围内执行真实订单。

前置条件：

- Phase 1-5 全部通过。
- 独立 runtime / broker write authorization。
- kill switch 可用且演练通过。
- broker adapter 只允许 OMS 调用。

阶段目标：

| 能力 | 目标 |
|---|---|
| Broker adapter | submit / cancel / query 全部有审计 |
| OMS state machine | order lifecycle 可追踪 |
| Pre-trade risk | 单笔金额、单票权重、行业暴露、涨跌停、停牌、黑名单 |
| Kill switch | 全局停止、策略停止、撤单、禁用 broker write |
| Reconciliation | broker position/cash vs internal ledger |
| Post-trade audit | 每笔订单追溯到 strategy artifact、signal、risk decision |
| Manual override | 人工暂停、恢复、降级 |

实盘等级：

| 等级 | 范围 |
|---|---|
| L1 readonly live | 不下单 |
| L2 paper / shadow | 模拟全链路 |
| L3 small-live | 小资金、白名单、严格限额 |
| L4 controlled-live | 扩大资金和标的，但仍有人审 |
| L5 production-live | 自动运行，但保留监控和 kill switch |

量化退出条件：

| 条件 | 门槛 |
|---|---|
| Evaluation window | >=20 个交易日 small-live |
| Reconciliation | 每日 position / cash reconciliation 失败数 = 0；若失败必须当天人工关闭 |
| Order audit | 100% 真实订单可追溯到 strategy artifact、signal、risk decision、broker response |
| Kill switch drill | 至少 2 次演练 PASS：strategy-level stop 和 global stop |
| Risk breach | 未审批 risk breach = 0 |
| Incident runbook | 100% P0/P1 incident 类型有 runbook 和 owner |

### Phase 7：正式生产运行

目标：让策略从“能跑”变成“可长期运营”。

阶段目标：

| 能力 | 目标 |
|---|---|
| Daily runbook | 开盘前、盘中、收盘后流程 |
| Monitoring | 数据延迟、行情缺失、订单异常、PnL 异常、风险超限 |
| Alerting | 本地 / webhook / 邮件 / 其他告警 |
| Incident response | broker 异常、行情异常、数据漂移、风控异常 |
| SLA | 数据更新、策略生成、下单窗口、对账窗口 |
| Post-trade analytics | 成交质量、滑点、归因、偏离 |
| Model drift | ML 特征漂移、预测分布漂移、性能衰减 |
| Strategy lifecycle | 上线、暂停、下线、回滚、再训练、再审批 |

量化退出条件：

| 条件 | 门槛 |
|---|---|
| Daily reports | 连续 >=60 个交易日生成 daily report，缺失数 = 0 |
| Ledger coverage | 100% production actions 有 ledger event |
| Alerting | P0/P1 alert delivery success rate = 100% in monthly drill |
| Incident response | P0 incident tabletop drill 每月 >=1 次，演练记录 100% 入库 |
| Model drift | ML 策略每个交易日生成 drift summary；缺失数 = 0 |

### Phase 8：平台化与多策略组合

目标：从单策略生产运行扩展到多策略组合平台。

阶段目标：

| 能力 | 目标 |
|---|---|
| Capital allocation | 多策略资金分配 |
| Risk budget | 策略级 / 组合级风险预算 |
| Correlation management | 策略相关性监控 |
| Order netting | 多策略订单冲突净额化 |
| Capacity management | 策略容量和冲击成本管理 |
| Portfolio-level drawdown control | 组合层止损 / 降杠杆 |
| Strategy retirement | 策略退役、暂停、降级 |

量化退出条件：

| 条件 | 门槛 |
|---|---|
| Strategy count | >=3 个策略可在 paper 或 controlled-live 中并行运行 |
| Risk budget | 100% 策略有 strategy-level 和 portfolio-level risk budget |
| Order netting | 100% 多策略冲突订单经过 netting / conflict policy |
| Portfolio controls | 组合级 drawdown / exposure breach 触发降级或停止，漏报数 = 0 |
| Audit | 资金分配、订单净额化、策略暂停 100% 有 ledger |

## 5. 推荐 CR 切分

建议不要直接启动“实盘平台 CR”。推荐先以 parent CR 方式分批推进。

| CR | 名称 | 范围 |
|---|---|---|
| CR-147 | Quant Research Production Foundation | 已完成：ResearchDatasetSpec、feature/label/training snapshot、experiment/admission 资产收敛 |
| CR-148 | Unified Backtest and Experiment Foundation | 已完成：BacktestRunSpec、report pack、cost/risk/attribution metadata contract |
| CR-149 | Governed Lake Readiness Matrix Foundation | 已完成本地 Phase 1 治理；NAS scoped sync deferred 为 `RA-CR149-001` |
| CR-150 | Multifactor Framework Completion | 当前 active：补齐日频多因子 baseline framework metadata chain |
| Future CR | Paper Trading and Readonly Live | readonly market/account、live signal shadow、paper OMS、daily paper report |
| Future CR | Controlled Small-Live Authorization Gate | OMS、broker adapter write contract、pre-trade risk、kill switch、小额实盘门禁 |

### CR-147 推荐范围

CR-147 应优先做研究生产底座，而不是实盘。

In scope：

1. Dataset readiness matrix。
2. Conflict policy for `trade_status` / `prices_limit` / `prices` / `events` / `adj_factor`。
3. PIT readiness contract。
4. Run registry / source_run_id governance。
5. `ResearchDatasetSpec`。
6. PIT-safe dataset builder。
7. Baseline multi-factor backtest。
8. Experiment registry 最小闭环。
9. NAS multi-node consistency design and read-only / dry-run gate。
10. 既有 FeatureArtifactSpec / ExperimentManifest / StrategyAdmissionPackage / paper simulation 资产衔接计划。

Out of scope：

1. 真实下单。
2. broker write。
3. provider fetch 自动化。
4. NAS sync。
5. N1 physical migration。
6. ML production training。
7. event live trading。
8. minute / Level2 / Qlib / Backtrader 深度集成，除非另起 deferred capability CR。

## 6. 风险与门禁

| 风险 | 门禁策略 |
|---|---|
| business-conflict 自动择优污染历史事实 | 必须 dataset-level human gate |
| ML 数据泄漏 | ResearchDatasetSpec + leakage gate 必须先通过 |
| 事件策略使用未来事件 | event_time / available_at / decision_time 必须分离 |
| 回测与 live 口径不一致 | readonly live replay reconciliation |
| 策略绕过风控直接下单 | 禁止 Strategy 直接调用 broker adapter |
| 小额实盘越权 | 单独 runtime_authorization + broker write gate |
| kill switch 不可用 | 不允许进入 small-live |
| NAS 多节点视图不一致 | 不允许进入 Phase 5 readonly live / paper |
| published pointer 未前移或多节点 checksum 不一致 | 不允许宣称 production shared current truth |

## 7. 近期建议

近期最优先不是 N1 physical migration，也不是实盘。

建议顺序：

1. CR146 CP2 approve 后关闭当前 validation delivery。
2. 启动 CR147。
3. 先治理 `trade_status` conflict policy。
4. 再治理 `prices_limit` / `prices`。
5. 建立 PIT readiness matrix。
6. 实现 `ResearchDatasetSpec` 和 PIT-safe builder。
7. 以日频多因子 baseline 打通 research -> backtest -> experiment registry，明确 baseline 是链路验证，不是策略锁定或上线承诺。
8. 再扩展 ML 和事件驱动。
9. 最后进入 readonly live / paper / small-live。

核心原则：

> 先让研究结果可信，再让回测可比较，再让 paper/live 可对账，最后才让真实订单可执行。

## 8. Gotchas

1. **不要把 baseline 多因子误读为策略方向锁定。** Phase 3 的多因子 baseline 是验证 research -> backtest -> registry 链路，不是承诺先上线多因子策略。
2. **不要把 quarantine policy 误读为冲突清零。** Phase 1 的目标是让 4,272,624 个 business-conflict groups 有处理状态；未授权语义择优前，正确结果是 quarantine / fail-closed，而不是自动挑主记录。
3. **不要推迟 NAS 一致性到实盘前夜。** 多节点 shared-read、published pointer、snapshot hash 一致性是 Phase 5 前置条件，不是 Phase 6 附加项。
4. **不要从零重建已有资产。** Feature Store、Experiment registry、Strategy artifact、Paper simulation、Backtest 在仓库已有基础实现；后续 CR 应做收敛、契约和证据，而不是平行实现。
5. **不要把 live readonly 视为低风险小事。** 即使不下单，live signal 与 offline replay 不一致也会暴露数据时钟、特征口径或运行环境问题。
6. **不要让 Strategy 直接产生 broker order。** Strategy 只能输出 signal / target；真实订单必须由 OMS 经 pre-trade risk 和 kill switch 控制。
7. **不要让事件策略使用 event_time 当作可交易时间。** 事件驱动必须同时记录 event_time、available_at 和 decision_time，否则回测会系统性高估。
8. **不要把 N1 physical migration 放在数据事实治理之前。** 先治理 conflict policy、PIT contract 和 run registry，再考虑物理迁移。
