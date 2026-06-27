---
context_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-PHASE-GOALS-AND-PLAN-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T11:10:00+08:00"
owner: "host-orchestrator"
status: "active-context-entry"
current_phase: "story-execution"
runtime_authorization_granted: false
simulation_runtime_authorization_required_per_run: true
small_live_or_live_authorized: false
updated_at: "2026-06-26T14:20:00+08:00"
---

# Runner QMT Simulation Multifactor Next Phase Goals And Plan

## 0. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1 | 2026-06-26 | host-orchestrator | 建立 r3 后续阶段目标，后续推进到 5/5 readiness closure。 |
| v2 | 2026-06-26 | host-orchestrator | 根据用户确认，重新定义下一阶段目标：模拟盘仍为目标，但必须从 runner 注入包样例升级为可解释、可审计、可扩展到真实股票池的成熟多因子模拟盘策略；`small_live` / `live` 准备后移。 |
| v3 | 2026-06-26 | host-orchestrator | 根据用户最新阶段目标，将后续路线重定义为五阶段：Stage 1 刷新 `quant-lab` 项目蓝图；Stage 2 在不连接数据湖的前提下升级多因子研究框架；Stage 3 在研究机连接数据湖生产成熟多因子模拟盘策略；Stage 4 模拟盘运行和观察；Stage 5 小额实盘和实盘运行。 |
| v4 | 2026-06-26 | host-orchestrator | 启动 Stage 2 no-lake 初始实现切片：新增成熟多因子框架合同、typed unavailable、ResearchEvidenceIndex、PortfolioRiskPolicy、MatureAdmissionSupport、fixture 和测试；未连接数据湖、未执行 runtime。 |
| v5 | 2026-06-26 | host-orchestrator | 完成 Stage 2 no-lake 桥接切片：正式打通 MatureAdmissionSupport 与 CR030 / CR039 输出，统一项目级 StrategyCandidate，并补 Stage 3 研究机交接合同；未连接数据湖、未执行 runtime。 |

## 1. 目的

本文档作为后续恢复入口，统一记录 `RUNNER-QMT-SIMULATION-MULTIFACTOR` 在交易窗口验收和 `5/5` readiness closure 后的阶段目标、计划、授权边界和下一步执行顺序。

它服务四个后续方向：

1. 保留已完成的 runner / gateway / simulation submit-cancel readiness 结论。
2. 将下一阶段入口从单纯 runner runtime 继续验收，切换为 `quant-lab` 项目级蓝图刷新。
3. 将成熟多因子策略生产拆成 Stage 2 框架升级和 Stage 3 真实数据策略生产，避免把数据湖连接提前引入框架升级阶段。
4. 在成熟策略通过研究、风控、准入和 simulation 观察门禁后，再为未来 `small_live` / `live` 切换准备独立 CR 和决策输入。

本文档不是 runtime authorization，不授权继续下单、撤单、读取凭据、读取 raw account / raw order / fund detail，也不授权 `small_live` / `live`。

## 1.1 目标重定义

截至 v3，下一阶段目标不再是继续证明 runner 能运行脱敏 fixture / runtime overlay 样例，也不直接进入成熟多因子策略生产。该能力已经由 `5/5` simulation runtime evidence 和 `READY_WITH_RISK` readiness closure 证明。

新的进入顺序是：

1. 先把 `quant-lab` 项目蓝图刷新为统一项目蓝图，而不是新增一份孤立的端到端策略研究蓝图。
2. 再在蓝图基础上升级多因子研究框架。该阶段不连接数据湖，只完成框架、合同、fixture、schema、测试和 typed unavailable 处理能力。
3. 然后在研究机连接数据湖，生产一条真正可长期模拟盘观察的成熟多因子策略。
4. 再进入模拟盘运行和观察，按日常证据与调整判断是否达到实盘入口条件。
5. 最后通过独立 live switch CR 进入小额实盘和实盘运行。

新的目标是形成一个真正适合进入长期模拟盘观察的多因子策略：

- 可解释：因子定义、方向、经济含义、计算窗口、可用时点和异常值处理可追踪。
- 可审计：数据 lineage、factor panel、label window、单因子评价、多因子合成、组合生成、风险限制、准入包和 simulation evidence 均有路径引用。
- 可扩展：选股范围从 2 个 ETF / fixture 样例扩展到明确的真实股票池，并定义 universe 进入 / 退出规则、停牌 / ST / 流动性 / 涨跌停 / 可交易性过滤。
- 可运行：成熟策略准入包可由 runner 导入，通过 P0/P0.5/P1/P2/P3/P4，并能按逐次授权进入 simulation runtime。
- 可长期观察：具备模拟盘运行日历、调仓频率、观察指标、日报摘要、风险暴露监控、异常处理和停止条件。

因此，当前 `READY_WITH_RISK` 只表示 runner / gateway / simulation operational readiness；不表示当前 fixture 策略已经达到成熟多因子策略标准。

## 2. 当前状态快照

| 项目 | 状态 |
|---|---|
| Meta Flow phase | `story-execution` |
| Blocked | `false` |
| Active CR | none |
| Active Story | none |
| CR138 fixture/static baseline | CP8 已以 `READY_WITH_RISK` 关闭 |
| 真实 QMT simulation runtime | r3 交易窗口验收已通过 |
| Stability window | `completed_runs=5/5`，`pass` |
| `small_live` / `live` | deferred，未授权、未实现、未验证 |
| 成熟多因子策略 | not_ready，当前仅有 fixture / 综合 alpha score 样例 |
| 下一阶段目标 | stage_1_quant_lab_project_blueprint_refresh |

### 2.1 r3 runtime 结论

| 阶段 | 结论 | 关键结果 |
|---|---|---|
| P0 gateway health / identity | PASS | `runtime_mode=simulation`、`account_kind=simulation`、`runtime_profile=cr138-simulation`、`session_ready=true` |
| P0.5 signed positions readonly | PASS | positions 条数为 0，evidence 脱敏 |
| P1 target | PASS | target 生成通过 |
| P2 order plan | PASS | `order_count=1` |
| P3 execution | PASS | simulation submit/cancel 各 1 次，`unknown_count=0` |
| P4 reconciliation | PASS | `unresolved_count=0`、`manual_takeover_required=false` |

### 2.2 关键证据

| 类型 | 路径 |
|---|---|
| r3 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R3-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r3 readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r3.json` |
| r3 operator evidence | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.json` |
| r3 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.index.json` |
| r4 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R4-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r4 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r4.json` |
| r4 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r4/operator-evidence.index.json` |
| r5 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R5-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r5 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r5.json` |
| r5 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r5/operator-evidence.index.json` |
| r6 runtime acceptance | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R6-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| r6 stability window summary | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r6.json` |
| r6 operator evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r6/operator-evidence.index.json` |
| Phase B readiness closure | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-OPERATIONAL-READINESS-CLOSURE-2026-06-26.md` |
| Phase B release context | `process/release/RELEASE-CONTEXT-RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-READINESS-2026-06-26.yaml` |
| Phase B human decision | `process/checkpoints/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-READINESS-HUMAN-DECISION-2026-06-26.md` |
| Ops policy | `process/policies/RUNNER-QMT-SIMULATION-MULTIFACTOR-RUNTIME-INPUT-GATEWAY-LIFECYCLE-POLICY-2026-06-26.md` |
| Ops policy closure | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-OPS-POLICY-CLOSURE-2026-06-26.md` |
| deferred live switch scenario pack | `process/context/RUNNER-QMT-SIMULATION-TO-LIVE-SWITCH-DEFERRED-SCENARIO-PACK-2026-06-25.md` |

## 3. 后续阶段目标

### Stage 1: Quant-Lab Project Blueprint Refresh

目标：在当前 `docs/design/BLUEPRINT.md`、`docs/design/DOMAIN-MAP.md` 和 `docs/design/DEPENDENCY-MAP.md` 基础上刷新 `quant-lab` 项目蓝图。蓝图不是单独的端到端策略研究蓝图，而是整个项目的能力边界、领域对象、依赖方向和阶段路线入口。

进入条件：

- Phase A / B runner / gateway / simulation operational readiness 已关闭为 `READY_WITH_RISK`，且用户已接受。
- 用户明确要求在当前蓝图基础上修改、合并，并可将原蓝图归档。
- 本阶段不执行 runtime、不连接数据湖、不启动 gateway。

完成条件：

- 原蓝图三件套已归档到 `process/archive/design-blueprints/`。
- `docs/design/BLUEPRINT.md` 已明确五阶段路线：
  - Stage 1：项目蓝图刷新。
  - Stage 2：多因子研究框架升级，不连接数据湖。
  - Stage 3：在研究机连接数据湖，生产成熟多因子模拟盘策略。
  - Stage 4：模拟盘运行、观察和日常调整。
  - Stage 5：小额实盘和实盘运行。
- `docs/design/DOMAIN-MAP.md` 已补齐多策略类型适配、Stage 2 框架升级、Stage 3 成熟策略、Stage 4 模拟盘观察领域对象。
- `docs/design/DEPENDENCY-MAP.md` 已明确 Stage 2 no-lake、Stage 3 data-lake-on-research-machine、Stage 4 per-run simulation authorization、Stage 5 independent live switch CR。
- `process/STATE.md` 和 `process/state/STATE.current.json` 指向 Stage 1 关闭后的下一步，即 Stage 2 多因子研究框架升级。

非目标：

- 不把蓝图改成只服务多因子的策略研究文档。
- 不启动数据湖连接。
- 不生成成熟策略。
- 不执行 simulation runtime。

### Stage 2: Multifactor Research Framework Upgrade

目标：在 Stage 1 蓝图基础上升级 `quant-lab` 中的多因子研究框架，使其能够支撑 Stage 3 的成熟策略生产。

当前实现状态：`bridge_slice_done`。已完成 Stage 2 no-lake 的基础合同、CR030 / CR039 输出桥接、项目级 `StrategyCandidate` 统一和 Stage 3 研究机交接合同，但还未进入 Stage 3 真实策略生产。

数据边界：

- 本阶段不连接数据湖。
- 允许使用 fixture、schema、静态样例、mock current truth、typed unavailable 和合成小样本验证框架合同。
- 不允许 provider fetch、lake write、catalog publish、真实数据湖 current pointer 修改或任何凭据读取。

完成条件：

| 条件 | 状态 | 证据 / 备注 |
|---|---|---|
| 多因子研究框架能表达真实策略需要的 `FactorSpec`、`FactorRunSpec`、`FactorPanelContract`、`LabelWindowSpec`、`FactorEvaluationReport`、`MultiFactorCombiner`、`PortfolioRiskPolicy` 和 mature `StrategyAdmissionPackage` | pass-stage2 | 旧 CR030 对象已存在；Stage 2 新增 `PortfolioRiskPolicy`、`MatureAdmissionSupport` 和 `Stage2MatureFrameworkBundle`。 |
| 框架能记录每个因子的输入字段、计算窗口、available_at、缺失处理、winsorize / neutralize / zscore 规则和 lineage placeholder | partial | `FactorSpec` 已支持；Stage 2 fixture 覆盖 no-lake lineage placeholder。 |
| 框架能将多因子输出归一到项目级 `SignalSet` / `StrategyCandidate` / `ResearchEvidenceIndex` | pass-stage2 | `build_project_strategy_candidate_from_cr039()` 已将 CR039 候选正式归一到 `strategy_candidate_v1`，并绑定 `SignalSet`、`ResearchEvidenceIndex` 和 `PortfolioRiskPolicy`。 |
| 框架能为 Stage 3 返回明确的数据需求清单，包括 PIT universe、上市退市、ST、停牌、涨跌停、流动性、行业 / 市值 / 风格、benchmark、费用 / 滑点等 | pass-stage2 | `MatureAdmissionSupport.stage3_data_requirements` 与 `Stage3ResearchMachineHandoff.required_inputs` 覆盖 Stage 3 数据需求。 |
| 框架能为 Stage 3 返回明确 evidence 要求，包括 factor panel、label window、IC / RankIC、分层收益、换手、暴露、组合版本、风险版本、准入包和 runner preflight | pass-stage2 | `Stage3ResearchMachineHandoff.required_evidence` 已固化研究机 evidence 清单。 |
| 自动化测试或等价验证证明：在无数据湖情况下，缺失真实数据时返回 typed unavailable，而不是伪造 evidence pass | pass-stage2 | `tests/test_stage2_mature_multifactor_framework.py` 覆盖 typed unavailable、no-lake counter、CR039 候选统一、bundle 和 Stage 3 handoff。 |

当前 Stage 2 初始产物：

| 类型 | 路径 |
|---|---|
| Stage 2 framework module | `engine/mature_multifactor_framework.py` |
| Stage 2 tests | `tests/test_stage2_mature_multifactor_framework.py` |
| Stage 2 fixture | `tests/fixtures/stage2_multifactor_framework/mature_multifactor_stage2_fixture.json` |
| Stage 2 check | `process/checks/STAGE2-MULTIFACTOR-FRAMEWORK-NO-LAKE-INITIAL-SLICE-2026-06-26.md` |

非目标：

- 不生产真实策略。
- 不读取真实数据湖。
- 不进入 runner simulation。
- 不把 fixture 指标解释为真实研究结论。

### Stage 3: Production Mature Multifactor Strategy

目标：在研究机上连接数据湖，生产一条可解释、可审计、可扩展到真实股票池、可长期模拟盘观察的多因子策略。

数据边界：

- 本阶段可以连接数据湖，由用户在研究机上实施。
- 必须记录数据 release、run manifest、config hash、factor version、signal version、portfolio version、risk policy version 和 evidence index。
- 不授权 `small_live` / `live`，也不自动授权 QMT runtime。

策略要求：

- 可解释：因子必须有真实输入字段、计算窗口、可用时点、处理规则、方向和经济含义。
- 可审计：factor panel、label window、IC / RankIC、分层收益、换手、成本、暴露、组合构造和风险过滤必须有 lineage。
- 可扩展：真实股票池必须有 PIT universe、上市退市、ST、停牌、涨跌停、流动性、行业 / 市值 / 风格等数据支持。
- 可长期观察：每日或每次调仓的输入版本、信号版本、组合版本、风险版本和运行证据必须可追溯。

完成条件：

- 生成 mature `StrategyAdmissionPackage`，且不包含 raw account、token、raw order、raw broker ref、未脱敏私有路径或 live-ready 声明。
- runner offline / plan-only / preflight-only 可导入 mature admission package。
- 形成进入 Stage 4 的 simulation observation plan。

### Stage 4: Simulation Operation And Observation

目标：完成模拟盘运行，并观察模拟盘运行情况，通过日常调整使其达到实盘入口条件。

完成条件：

- 每次 simulation runtime 都取得新的逐次授权，并从 P0 health / runtime identity 开始。
- 每次运行 evidence 脱敏，记录输入版本、信号版本、组合版本、风险版本和 run evidence index。
- 形成固定观察周期和指标：收益、回撤、换手、暴露、成交状态、reconciliation、unknown order、manual takeover、异常分类和 stop condition。
- 日常调整必须记录到 StrategyChangePlan 或观察计划，不允许未追踪参数漂移。
- 若达到 small_live 候选条件，输出 live switch CR 输入，而不是自动进入实盘。

### Stage 5: Small Live And Live Operation

目标：在 Stage 4 被接受后，通过独立 live switch CR 进入小额实盘运行，再按门禁评估是否升级实盘。

完成条件：

- 创建独立 live switch CR 或 change package。
- 明确 `small_live` 的 symbol、side、quantity、notional、frequency、max order count、max loss、kill switch、cancel-only rollback、manual takeover 和 incident path。
- 明确 live gateway profile、授权窗口、证据脱敏、风险接受和回滚策略。
- `small_live` 通过后，scale live 仍需独立人工决策，不自动升级。

非目标：

- 不复用 simulation runtime authorization。
- 不把 Stage 4 observation pass 解释为 live approval。

## 3.1 已完成历史阶段：Runner / Gateway Simulation Readiness

### Phase A: Simulation Stability Window Completion

目标：把 simulation runtime 稳定性窗口补到 `5/5`。当前已完成。

完成条件：

- 5 次独立 simulation runtime 成功记录已具备。
- 每次执行前都取得新的逐次 `simulation runtime authorization`。
- 每次都从 P0 gateway health / runtime identity 开始。
- 每次 signed positions readonly 通过。
- 每次 P1 / P2 / P3 / P4 通过。
- 每次 `unknown_count=0`。
- 每次 `unresolved_count=0`。
- 每次 `manual_takeover_required=false`。
- 每次 evidence 脱敏扫描通过。
- 每次都记录 evidence index 和验收摘要。

非目标：

- 不启动 `small_live` / `live`。
- 不复用上一轮 runtime authorization。
- 不保存 raw account、raw symbol、raw broker order ref、raw payload、secret/token 或 fund detail。
- 不触达 NAS、provider/lake/catalog、publish 或 remote Git。

### Phase B: Simulation Operational Readiness Closure

目标：在 stability window 达到 `5/5` 后，把 simulation runtime readiness 收口为可审计结论。

完成条件：

- 汇总 `5/5` simulation runtime evidence。已完成。
- 生成或更新 simulation runtime verification summary。已完成。
- 更新 `process/STATE.md` 和 `process/state/STATE.current.json`。已完成。
- 更新或补充 operational runbook。已完成。
- 明确 simulation readiness 的范围和非范围。已完成。
- 明确仍不包含 `small_live` / `live`。已完成。
- 必要时发起 CP7 / CP8 等价人工确认或 release readiness 更新。当前结论为 `READY_WITH_RISK`，且用户已接受；第三个交易日观察保留为可选后续，不再阻塞 simulation readiness closure。
- 固化 `SIM-OPS-GATE-02` runtime input policy。已完成。
- 固化 `SIM-OPS-GATE-03` gateway lifecycle policy。已完成。

非目标：

- 不把 simulation readiness 自动升级为 live readiness。
- 不创建或启用 live gateway route。
- 不读取 live credential。
- 不触达真实资金相关 raw detail。

### Historical Phase C: Mature Multifactor Strategy Simulation Readiness Superseded By Stage 2 And Stage 3

目标：将当前可运行的 runner 注入包样例升级为可解释、可审计、可扩展到真实股票池、可长期观察的成熟多因子模拟盘策略。

进入条件：

- Phase A / B 已完成，runner / gateway / simulation operational readiness 已收口为 `READY_WITH_RISK`。
- 用户明确下一阶段目标仍是 simulation，不进入 `small_live` / `live`。
- 本阶段不执行 runtime，除非后续单独取得逐次 `simulation runtime authorization`。

完成条件：

- Strategy Research Definition：
  - 明确策略名称、目标、持有周期、调仓频率、benchmark、目标市场和运行假设。
  - 明确选股范围：真实股票池定义、PIT 规则、成分变更、上市天数、ST / 停牌 / 退市 / 涨跌停 / 流动性过滤。
  - 明确不再以 2 个 ETF / fixture 作为成熟策略 universe。
- Factor Definition：
  - 至少形成一版可审计 `FactorSpec` 清单，逐项记录因子名称、方向、经济含义、输入字段、计算窗口、available_at、缺失处理、winsorize / neutralize / zscore 规则。
  - 禁止只用 `alpha_score` 或 `fixture:factor-panel` 作为成熟策略解释。
- Research Evidence：
  - 生成 factor panel、label window、no-lookahead 检查、单因子 IC / RankIC / 分层收益 / turnover / cost / exposure 报告。
  - 形成多因子合成规则，例如 `rule_weight`、`linear_score` 或明确的模型公式。
  - 记录样本内 / 样本外或滚动窗口评价；不可用指标必须给出 typed unavailable 原因。
- Portfolio And Risk：
  - 形成 `MultiFactorPortfolioPlan`，定义 `top_n`、max weight、行业 / 风格暴露、单票上限、组合上限、换手上限、容量 / 流动性假设、费用和滑点假设。
  - 明确 simulation order plan 的资金基准、lot size、price deviation、fee buffer、stop-new-orders、manual takeover 和 cancel-only rollback。
- StrategyAdmissionPackage：
  - 生成非 fixture 的成熟策略准入包，包含数据证据、因子证据、组合证据、风险证据、blocked claims 和 limitations。
  - 准入包不得包含真实账号、token、raw order、raw broker ref、未脱敏私有路径或 live-ready 声明。
- Simulation Dry Run And Runtime Gate：
  - 先通过 offline / plan-only / preflight-only 检查，证明 runner 可导入成熟策略包。
  - 只有用户逐次授权后，才能进入 P0/P0.5/P1/P2/P3/P4 simulation runtime。
  - 初期成熟策略 runtime 仍以小规模、低频、可人工接管为准；运行 evidence 必须脱敏。
- Long Observation Readiness：
  - 定义长期模拟盘观察周期、每日 / 每次调仓检查项、收益 / 回撤 / 换手 / 暴露 / 成交状态 / 异常分类指标。
  - 定义停止条件：数据质量失败、风控失败、gateway identity 不一致、reconciliation unresolved、unknown order status、暴露超限、人工接管触发。

非目标：

- 不把 runner fixture 样例当作成熟策略。
- 不在未完成因子定义和真实股票池审计前启动长期模拟盘。
- 不自动继续 submit/cancel。
- 不创建或启用 `small_live` / `live` route。
- 不把 simulation readiness 解释为 live readiness。

建议产物：

| 产物 | 建议路径 | 用途 |
|---|---|---|
| 成熟策略阶段计划 | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-MATURE-STRATEGY-SIMULATION-PLAN-2026-06-26.md` | 下一阶段执行入口 |
| FactorSpec 清单 | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FACTOR-SPECS-2026-06-26.md` 或后续正式 docs 路径 | 因子解释和审计 |
| Universe 定义 | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-UNIVERSE-POLICY-2026-06-26.md` | 真实股票池边界 |
| Research evidence index | `process/evidence/...` | factor / backtest / portfolio 证据索引 |
| 成熟策略准入包 | 私有或脱敏可审计路径 | runner 导入输入 |
| Simulation observation runbook | `process/runbooks/...` | 长期模拟盘观察流程 |

### Historical Phase D: Small Live / Live Switch Preparation Superseded By Stage 5

目标：为未来 `small_live` / `live` 切换准备独立 CR 和人工决策输入。

进入条件：

- Stage 4 成熟多因子模拟盘观察已完成，并通过用户接受的 simulation observation readiness。
- `small_live` / `live` 仍需独立 CR、独立人工门禁和独立 runtime authorization。

完成条件：

- 创建独立 live switch CR 或 change package。
- 将 `DQ-LIVE-001..004` 转成正式人工决策项。
- 明确 live gateway 是否使用独立 profile / port / env。
- 明确实盘前需要的 simulation stability window 和审批门槛。
- 明确首个实盘范围只能是 `small_live`，且包含 symbol、side、quantity、notional、frequency、max order count 等限制。
- 明确 live evidence 只允许保存脱敏 summary、count、digest、ref 和风险接受记录。
- 明确 cancel-only rollback、stop-new-orders、manual takeover 和 incident path。

非目标：

- 不在本文档内实现 live submit/cancel。
- 不在本文档内激活 `small_live` / `live` route。
- 不把 simulation authorization 复用为 live authorization。
- 不把 r3 PASS 或未来 `5/5` simulation PASS 解释为 live approval。

## 4. 执行顺序

| 顺序 | 任务 | 触发条件 | 产物 |
|---|---|---|---|
| 1 | Phase A r4 runtime authorization | 已完成 | r4 P0-P4 evidence、acceptance check |
| 2 | Phase A r5 runtime authorization | 已完成 | r5 P0-P4 evidence、acceptance check |
| 3 | Phase A r6 runtime authorization | 已完成 | r6 P0-P4 evidence、acceptance check |
| 4 | Stability window summary | 已完成，`completed_runs=5/5` | simulation stability window summary |
| 5 | Phase B readiness closure | 已形成且用户已接受 `READY_WITH_RISK` | readiness closure / release context / runbook addendum / human decision checkpoint |
| 6 | SIM-OPS-GATE-02/03 policy closure | 已完成 | ops policy / ops policy closure check |
| 7 | Stage 1 quant-lab project blueprint refresh | 当前用户已确认五阶段路线 | 更新 `BLUEPRINT` / `DOMAIN-MAP` / `DEPENDENCY-MAP`，归档旧蓝图，刷新 STATE |
| 8 | Stage 2 multifactor research framework upgrade | 已完成 no-lake 桥接切片；不连接数据湖 | bridge slice done：framework contract / CR030-CR039 bridge / project StrategyCandidate / Stage 3 handoff / test evidence |
| 9 | Stage 3 production mature multifactor strategy | Stage 2 handoff 已具备；用户在研究机连接数据湖 | real-data FactorSpec / factor panel / label window / IC-RankIC / portfolio-risk / mature admission package / runner offline preflight |
| 10 | Stage 4 simulation operation and observation | mature admission + runner offline/preflight 通过；用户逐次 runtime 授权并启动 gateway | P0-P4 simulation evidence / observation report / StrategyChangePlan |
| 11 | Stage 5 small_live / live | Stage 4 observation 被接受，且用户明确要进入实盘准备 | live switch CR / small_live decision brief / risk acceptance |

## 5. 授权边界

| 操作 | 当前状态 | 规则 |
|---|---|---|
| signed positions readonly | 需要逐次 runtime authorization | 每次执行前确认授权和 gateway identity |
| simulation submit/cancel | 需要逐次 runtime authorization | 每次只执行授权范围内的 runtime，不自动重试 |
| `small_live` / `live` | 未授权 | 必须独立 CR、独立人工决策、独立 runtime authorization |
| gateway 启停 | 用户手工管理 | 如需启动或重启，由用户在 Windows 执行 |
| 修改 Windows runtime 项目 | 条件允许 | 若改动 `/mnt/c/quant-lab-runtime/trading/*`，需要提示用户重启 gateway |
| NAS / provider / lake / catalog / publish / remote Git | 未授权 | 不得触达 |
| Stage 2 数据湖连接 | 未授权 | Stage 2 只升级框架，不连接数据湖 |
| Stage 3 数据湖连接 | 待后续授权 / 用户在研究机执行 | 仅用于真实策略生产，必须记录 release、lineage 和 evidence index |

## 6. 风险与阻断项

| 风险 ID | 状态 | 说明 | 处理 |
|---|---|---|---|
| RISK-STABILITY-WINDOW-INCOMPLETE | closed | simulation runtime 稳定窗口已达到 `5/5` | 进入 Phase B simulation readiness closure |
| RISK-RUNTIME-INPUT-OPERABILITY | policy_defined | r3 已用私有 overlay 通过；输入来源、刷新时机、字段边界和 evidence 脱敏策略已固化 | 若要自动化长期模拟盘，再实现 preflight checker |
| RISK-LIVE-MISREAD | controlled | r3 PASS 可能被误读为 live readiness | 本文档和 STATE 明确 live deferred |
| RISK-AUTHORIZATION-REUSE | controlled | 误复用上一轮 simulation authorization | 每次 runtime 前强制重新确认 |
| RISK-GATEWAY-DESYNC | policy_defined | WSL 仓库和 Windows runtime 项目可能不同步 | 已固化同步、重启和 P0/P0.5 复验规则 |
| RISK-THREE-TRADING-DAY-OBSERVATION | accepted_ready_with_risk | 5/5 count 已完成，但 runbook 保留 3 trading days 观察口径；当前证据主要覆盖两个日期 | 用户已接受 `READY_WITH_RISK`；如未来要求升级 `READY`，可补第三个交易日观察 |
| RISK-PROJECT-BLUEPRINT-DRIFT | mitigated | 若新增独立端到端策略研究蓝图，会造成 `quant-lab` 项目蓝图分叉 | Stage 1 已在当前蓝图三件套上合并路线，并归档旧蓝图 |
| RISK-STAGE2-DATA-LAKE-COUPLING | controlled | 多因子框架升级若过早连接数据湖，会把框架合同验证和真实数据环境耦合 | Stage 2 已通过 no-lake counter、typed unavailable、CR030/CR039 bridge 和 Stage 3 handoff 固化边界；真实数据生产推迟到 Stage 3 |
| RISK-MATURE-STRATEGY-GAP | open | 当前策略仍是 fixture / 综合 alpha score 样例，不具备完整因子解释、真实股票池、研究评价和长期观察标准 | Stage 2 已具备成熟策略框架和 handoff；Stage 3 仍需基于研究机数据湖生产真实策略研究和准入证据 |
| RISK-UNIVERSE-SCALE-UP | open | 从 2 个 ETF / fixture 扩展到真实股票池会引入数据质量、流动性、停牌、ST、涨跌停、容量和调仓复杂度 | Stage 3 必须按 handoff 定义 universe policy、风险过滤和证据追溯，不允许直接扩大 runtime |
| RISK-FACTOR-AUDITABILITY | controlled | 当前 runner runtime admission 只记录 `factor_panel_ref` 和 `alpha_score`，不足以解释成熟策略 | Stage 2 已定义 Stage 3 evidence 清单；真实 FactorSpec、factor evidence 和组合合成规则仍在 Stage 3 生产 |

## 7. 下一次恢复流程

下一次恢复本工作时，默认读取顺序：

1. 本文件。
2. `process/state/STATE.current.json`。
3. `docs/design/BLUEPRINT.md`、`docs/design/DOMAIN-MAP.md`、`docs/design/DEPENDENCY-MAP.md`。
4. Stage 2 多因子研究框架升级计划和后续产物。
5. 只有当 Stage 3 mature strategy admission package 和 offline / plan-only / preflight-only 检查完成，且用户明确授权单次 simulation runtime 时，才读取对应 runtime input 路径。
6. 向用户请求新的逐次 `simulation runtime authorization`。
7. 用户启动 Windows gateway 后，从 P0 health / runtime identity 开始。

恢复时禁止：

- 直接跳过 P0。
- 复用 r3 authorization。
- 自动继续 submit/cancel。
- 将 simulation 操作升级为 `small_live` / `live`。
- 把 fixture / 2 ETF 样例当作成熟多因子策略。
- 未按 Stage 2 handoff 准备输入和 evidence 清单就直接连接数据湖生产策略。
- 未完成 Stage 3 FactorSpec、universe policy 和 research evidence 就扩大到真实股票池 runtime。

## 8. 当前推荐下一步

`SIM-OPS-GATE-02` 和 `SIM-OPS-GATE-03` 已固化。不要在无新目标下继续重复 runtime submit/cancel。

当前用户已明确：下一步目标仍然是达到模拟盘，但不是运行注入包样例，而是形成可解释、可审计、可扩展到真实股票池的成熟多因子模拟盘策略。

因此当前推荐下一步不是直接启动 simulation runtime，也不是直接连接数据湖生产策略，而是按五阶段路线推进：

1. Stage 1：已按用户要求刷新 `quant-lab` 项目蓝图并归档旧蓝图。
2. Stage 2：已完成 no-lake 桥接切片，框架合同、CR030/CR039 bridge、项目级 StrategyCandidate 和 Stage 3 handoff 已具备。
3. Stage 3：下一步应按 handoff 清单，由用户在研究机连接数据湖生产成熟多因子策略。
4. Stage 4：成熟策略准入和 runner offline / plan-only / preflight-only 通过后，再逐次授权 simulation runtime 并进入模拟盘观察。
5. Stage 5：模拟盘观察达到实盘入口条件后，另行启动 live switch CR 进入小额实盘 / 实盘。

如果未来用户希望转向实盘准备，则不要复用 Stage 4 simulation 授权或证据；应另行启动独立 live switch CR，并把 `DQ-LIVE-001..004` 作为 CP2 / CP3 前的人工决策输入。
