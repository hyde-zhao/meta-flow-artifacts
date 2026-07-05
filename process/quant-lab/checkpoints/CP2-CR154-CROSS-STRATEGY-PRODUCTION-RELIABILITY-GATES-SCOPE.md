---
checkpoint_id: "CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE"
checkpoint_name: "CR154 Cross-Strategy Production Reliability Gates Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-02T23:25:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-02T23:58:00+08:00"
auto_check_result: "process/checks/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-154"
  artifacts:
    - "process/changes/CR-154-CROSS-STRATEGY-RELIABILITY-GATES-2026-07-03.md"
    - "process/context/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE-CONTEXT.yaml"
    - "process/USE-CASES.md"
    - "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md"
    - "docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md"
    - "docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md"
---

# CP2 CR154 Cross-Strategy Production Reliability Gates Scope Baseline 人工审查

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR154 formal CR exists | PASS | `process/changes/CR-154-CROSS-STRATEGY-RELIABILITY-GATES-2026-07-03.md` | `lifecycle_status=active`，`gate_status=cp2_pending`。 |
| Rule 41 precheck passed | PASS | `process/checks/CR154-RULE41-CONFLICT-PRECHECK-2026-07-02.md` | active formal CRs none、blocked formal CRs none、FU-CR152-001 不占锁。 |
| CP0 request intake passed | PASS | `process/checks/CP0-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-REQUEST-INTAKE.result.json` | CP0 decision PASS。 |
| CP1 use-case completeness passed | PASS | `process/checks/CP1-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-USE-CASE-COMPLETENESS.result.json` | 基于 UC-58 / UC-59 / UC-60、roadmap、remediation plan、E2E review 增量通过。 |
| CR153 baseline closed | PASS | `process/checks/CP8-CR153-EVENT-DRIVEN-STRATEGY-E2E-RELEASE-READINESS.result.json` | CR153 closed as READY_WITH_RISK；`R-CR154-DEFERRED-001` 被接受为后续 CR 输入。 |
| CR154 Scenario Gray Areas evidence exists | PASS | `process/discussions/CP2-CR154-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-CR154-DISCUSSION-CHECKPOINT.json` | 基于用户 startup request 记录 CR154 专属 SGQ 证据；CP2 最终批准等待用户明确 `approve`。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | CR154 是否只覆盖本地/static/fixture cross-strategy production reliability gates | PASS | `process/context/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE-CONTEXT.yaml` | 建议 approve。 |
| 2 | CP1 是否显式覆盖 UC-58 / UC-59 / UC-60 | PASS | `process/checks/CP1-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-USE-CASE-COMPLETENESS.result.json`、`process/USE-CASES.md` | 建议 approve。 |
| 3 | 横切 blocker 是否来自 roadmap / remediation plan / E2E review | PASS | `docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md#7-横切生产可靠性计划Future-CR154`、`docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md#4-横切生产实践评审三类策略共用` | 建议 approve。 |
| 4 | Backtest trap gate 是否进入待决策项 | PASS | `DQ-CP2-CR154-BACKTEST-TRAP-GATE` | 建议覆盖 lookahead、survivorship、data snooping、regime overfit、cost underestimation；CP3 必须把 multiple-testing / PBO / DSR 拆成可审计统计工件。 |
| 5 | Walk-forward / OOS / purged-embargo governance 是否进入待决策项 | PASS | `DQ-CP2-CR154-CV-GOVERNANCE` | 建议统一三类策略的 OOS / CV 语义。 |
| 6 | Survivorship-free / PIT universe gate 是否进入待决策项 | PASS | `DQ-CP2-CR154-SURVIVORSHIP-UNIVERSE-GATE` | 建议把 CR153 slot 升级为共享 gate contract。 |
| 7 | Capacity / impact / liquidity sizing 是否进入待决策项 | PASS | `DQ-CP2-CR154-CAPACITY-IMPACT-GATE` | 建议 first wave 只做 contract / status / refs，不做真实 TCA。 |
| 8 | Regime / attribution / reconciliation slots 是否进入待决策项 | PASS | `DQ-CP2-CR154-REGIME-ATTRIBUTION-RECONCILIATION-SLOTS` | 建议 slots only，不授权真实 runtime/broker reconciliation。 |
| 9 | CR151/CR152/CR153 admission gate 默认强制策略是否进入待决策项 | PASS | `DQ-CP2-CR154-ADMISSION-GATE-DEFAULT-POLICY` | 建议显式决定 default-required / release-blocking 条件和回退策略。 |
| 10 | No-runtime / no-real-data 边界是否明确 | PASS | `DQ-CP2-CR154-NO-RUNTIME-NO-REAL-DATA-BOUNDARY`、`process/changes/CR-154-CROSS-STRATEGY-RELIABILITY-GATES-2026-07-03.md#不授权范围` | 建议 approve no-runtime boundary。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP2 auto precheck PASS | PASS | `process/checks/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE.result.json` | 可发起人工确认。 |
| Pending decisions collected | PASS | 下方 Decision Brief | 7 项待用户确认。 |
| Context capsule ready | PASS | `process/context/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE-CONTEXT.yaml` | read_profile=compact。 |
| Unauthorized scope explicit | PASS | 下方 Decision Brief | approve 不授权任何 runtime、真实数据、真实 feed、broker/order、reconciliation、publish 或外部操作。 |
| Scenario Gray Areas handled | PASS | `process/discussions/CP2-CR154-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-CR154-DISCUSSION-CHECKPOINT.json` | standard CP2 SGQ 证据已记录。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Rule 41 precheck | `process/checks/CR154-RULE41-CONFLICT-PRECHECK-2026-07-02.md` | PASS | 冲突预检通过。 |
| CR154 formal CR | `process/changes/CR-154-CROSS-STRATEGY-RELIABILITY-GATES-2026-07-03.md` | PASS | 已创建并记录五维影响分析。 |
| CR154 summary | `process/changes/summaries/CR-154-CROSS-STRATEGY-RELIABILITY-GATES-2026-07-03.summary.json` | PASS | state-router / cr-tracking 轻量入口。 |
| CP0 result | `process/checks/CP0-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-REQUEST-INTAKE.result.json` | PASS | request intake。 |
| CP1 result | `process/checks/CP1-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-USE-CASE-COMPLETENESS.result.json` | PASS | 增量 UC-58 / UC-59 / UC-60 completeness。 |
| CP2 context capsule | `process/context/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE-CONTEXT.yaml` | PASS | compact profile。 |
| CP2 result JSON | `process/checks/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE.result.json` | PASS | 自动预检。 |
| CP2 scenario discussion log | `process/discussions/CP2-CR154-SCENARIO-DISCUSSION-LOG.md` | PASS | 记录 CR154 专属 Scenario Gray Areas 和 SGQ-CR154-001..005。 |
| CP2 discussion checkpoint | `process/checks/CP2-CR154-DISCUSSION-CHECKPOINT.json` | PASS | 讨论恢复点；`cp2_ready=true`，final human gate pending。 |
| CP2 human checkpoint | `process/checkpoints/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE.md` | pending | 等待用户回复。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR154 Cross-Strategy Production Reliability Gates 的 CP2 范围基线，让后续设计只围绕本地/static/fixture 的横切策略可靠性 gate 展开。 |
| 推荐动作 | `approve`：批准 CR154 Scope Baseline、first-wave 范围、no-runtime 安全边界、backtest trap gate、CV governance、PIT universe gate、capacity/impact gate、regime/attribution/reconciliation slots 和 admission default policy。 |
| approve 后会发生什么 | CR154 进入 CP3 设计；下一步会设计跨策略 gate 架构、状态语义、admission default policy、fixture/static 验证方案，并落实 `CP3-DC-CR154-001` 统计可靠性工件约束。 |
| approve 不授权什么 | 不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog/event-store/model-registry 写入；不授权 live event listener、真实事件 feed、真实下单、真实数据验证、真实 reconciliation、真实发布执行或源码实现。 |
| 不确认会阻塞什么 | 阻塞 CR154 的 CP3 设计、Story 拆解、LLD、实现、验证和 release readiness。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 CR154 summary、CR154 正文、UC-58/59/60、roadmap、remediation plan。 |
| 全文档读取扩展 | 2 次；用户明确要求读取 startup 文件、STATE、CR-INDEX、CR153 closure evidence、roadmap、remediation plan、E2E review；且 `docs/product/USE-CASES.md` 缺失后扩展读取 `process/USE-CASES.md`。 |
| 缺失 / waived 理由 | `docs/product/USE-CASES.md` 缺失；当前项目外置 process 中 `process/USE-CASES.md` 为已确认 UC-58/59/60 基线。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json.human_gate_decisions` | scanned | 0 | 0 | 启动前无 pending human decisions。 |
| Handoff | `process/handoffs/NEXT-SESSION-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-CP0-CP2-2026-07-02.md` | scanned | 7 | 7 | 全部纳入 CR154 CP2 required decisions。 |
| UC baseline | `process/USE-CASES.md#UC-58/59/60` | scanned | 4 | 4 | 横切 blocker 和 no-runtime boundary 进入决策或不授权项。 |
| Requested UC path | `docs/product/USE-CASES.md` | missing | 1 | 0 | 路径缺失已记录；使用 `process/USE-CASES.md` fallback，不作为用户决策项。 |
| Roadmap | `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` | scanned | 5 | 5 | CR154 split、auth boundary、deferred data-lake candidates。 |
| Remediation plan | `docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md` | scanned | 7 | 7 | Future CR154 能力清单、数据湖后置规则、推荐顺序。 |
| E2E review | `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` | scanned | 9 | 7 | XC-GAP-1..9 映射到 CP2 decisions 或 later-wave。 |
| CR153 closure evidence | `process/release/RELEASE-CONTEXT-CR153-EVENT-DRIVEN-STRATEGY-E2E.yaml` / CP8 result | scanned | 3 | 2 | `R-CR154-DEFERRED-001` 与 no-overclaim/no-runtime boundary 进入 CR154 scope 和 not-authorized items。 |
| CR154 Scenario Gray Areas | `process/discussions/CP2-CR154-SCENARIO-DISCUSSION-LOG.md` / `process/checks/CP2-CR154-DISCUSSION-CHECKPOINT.json` | scanned | 5 | 5 | SGQ-CR154-001..005 已记录问题、用户回答来源、复述确认和影响面。 |
| CP0 / CP1 / CP2 auto results | `process/checks/CP0-*`, `process/checks/CP1-*`, `process/checks/CP2-*` | scanned | 0 | 0 | PASS; no blockers. |
| 用户显式约束 | 当前对话 | scanned | 8 | 7 | local/static/fixture-only、no lake/NAS/provider/runtime/broker/credential/feed/order/reconciliation/publish 进入 DQ 或不授权项。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 7 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 1 | `DQ-CP2-CR154-NO-RUNTIME-NO-REAL-DATA-BOUNDARY` 确认本轮不授权真实运行/数据/交易/发布。 |
| agent 默认处理 | 3 | CP2 通过后的命名、fixture 文件布局、低风险字段命名由 agent 在 CP3/CP5 留证据处理。 |
| 仅审计记录 | 6 | CR151/152/153 closure、FU-CR152-001 candidate、CR-INDEX legacy warnings、deferred data-lake candidates、USE-CASES path fallback、repo SHA 对齐。 |

### CP3 硬性设计约束

| 约束 ID | 来源 | 约束 | CP3 验收要求 |
|---|---|---|---|
| CP3-DC-CR154-001 | 当前 CP2 review finding；`docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md#4-横切生产实践评审三类策略共用` | `DQ-CP2-CR154-BACKTEST-TRAP-GATE` 不得只落成 trap name / status 枚举；多重检验 / 数据偷窥 / PBO / DSR 必须是可审计统计工件。 | HLD / ADR 必须定义可机器验证的 statistical reliability artifacts，至少包含 multiple-testing correction refs、FDR/BH refs、White Reality Check / Hansen SPA refs、PBO / CSCV refs、DSR / Sharpe 或 IC deflation refs、trial count / effective trials、OOS split refs、purge / embargo refs、survivorship audit refs、impact / capacity refs、blocked claims 和 release-blocking reason；若某字段不适用，必须给出 `n/a-with-reason`。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR154-BACKTEST-TRAP-GATE | architecture | CR154 是否建立三类策略共享 backtest trap gate？ | 建立共享 gate，覆盖 lookahead、survivorship、data snooping、regime overfit、cost underestimation；CP3 强制拆出 multiple-testing correction refs、FDR/BH refs、White Reality Check / Hansen SPA refs、PBO / CSCV refs、DSR / Sharpe 或 IC deflation refs、trial count / effective trials、blocked claims 和 release-blocking reason；默认 fail-closed / needs-review。 | A. 仅写文档清单；B. 每类策略各自建 gate；C. 延后到真实 runtime 后再做。 | 推荐方案让核心回测陷阱和统计可靠性证据在本地 contract 层先可审计；A 不可机器验证；B 会重复且不一致；C 会让后续策略继续带隐性风险。 | 影响 research admission、backtest validation、statistical reliability artifacts 和 release blocking policy。 | 若 CP3 发现范围过大，可把 gate 拆成 trap registry + statistical artifact contract + per-strategy adapter 三阶段。 |
| DQ-CP2-CR154-CV-GOVERNANCE | architecture | Walk-forward / OOS / purged-embargo 是否应统一治理？ | 建立共享 CV governance，统一 rolling / OOS / purged / embargo status、refs、fail reasons 和 fixture semantics。 | A. 复用 CR152 ML CV 作为全部策略默认；B. 让 CR151/152/153 各自实现；C. 本轮不处理 CV。 | 推荐方案避免误用 ML 假设，也避免三套框架；A 容易污染事件/多因子语义；B 维护成本高；C 保留 blocker。 | 影响 CR151/152/153 admission 关系和 future Story 边界。 | 若某策略需要专项 CV，可在共享 contract 上加 strategy-specific adapter。 |
| DQ-CP2-CR154-SURVIVORSHIP-UNIVERSE-GATE | architecture | Survivorship-free / PIT universe 是否进入 CR154 first wave？ | 将 PIT universe / survivorship-free universe 从 CR153 slot 扩展为共享 gate contract；真实 universe 数据仍不授权。 | A. 完整实现真实 universe 构建；B. 只保留 CR153 slot；C. 只在文档提示。 | 推荐方案让风险机器可见且不触发真实数据；A 越权且范围过大；B/C 无法支撑 release-blocking。 | 影响样本有效性、PIT gate、strategy admission 和 no-overclaim wording。 | 若需要真实 universe 数据，另起 data authorization / production lake CR。 |
| DQ-CP2-CR154-CAPACITY-IMPACT-GATE | implementation | Capacity / market impact / liquidity sizing 如何进入 first wave？ | 定义 ADV participation、capacity dollars、impact model family、cost-underestimation status 和 liquidity sizing refs；first wave 只验证 contract semantics。 | A. 直接校准真实 TCA；B. 只沿用 commission/tax；C. 完全延后。 | 推荐方案补齐 XC-GAP-1/4 且不触发真实交易数据；A 需要真实成交/盘口授权；B 会继续低估成本；C 保留 blocker。 | 影响 backtest credibility、AUM scaling 和 release wording。 | 若后续有真实 execution data，再开 runtime/data gate 做校准。 |
| DQ-CP2-CR154-REGIME-ATTRIBUTION-RECONCILIATION-SLOTS | implementation | Regime / attribution / reconciliation 在 first wave 做到什么程度？ | 定义结构化 slots、status、refs 和 `n/a-with-reason`；不执行真实 runtime/broker reconciliation。 | A. 实现真实 live-vs-offline reconciliation；B. 不记录这些维度；C. 只写自由文本。 | 推荐方案保留生产可靠性接口且可静态验证；A 越权；B/C 不可审计。 | 影响 future paper/live readiness，但本轮不声明 readiness。 | 真实 broker/account reconciliation 必须另起 runtime authorization gate。 |
| DQ-CP2-CR154-ADMISSION-GATE-DEFAULT-POLICY | architecture | CR151/CR152/CR153 gates 何时从 opt-in / package-visible 变成 default-required 或 release-blocking？ | 定义 admission default policy，按 strategy class、release profile、risk level 和 evidence completeness 决定 default-required / release-blocking / opt-in exception。 | A. 立即全量强制；B. 继续全部 opt-in；C. 只由调用方自由选择。 | 推荐方案兼顾历史兼容和可靠性；A 可能破坏既有调用；B/C 会让新 gate 被绕过。 | 影响 CR150/151/152/153 调用方行为、completion map 和 release gate。 | 若 CP3 发现兼容风险高，可先只对新 release profile 强制。 |
| DQ-CP2-CR154-NO-RUNTIME-NO-REAL-DATA-BOUNDARY | security | CR154 CP2 approve 是否授权真实 runtime、真实数据、broker、feed、reconciliation 或 publish？ | 不授权。CR154 first wave 保持 local/static/fixture-only，真实外部系统和真实交易/发布动作必须另开人工 gate。 | A. 同时做真实 lake readonly validation；B. 同时做 paper/live reconciliation；C. 同时连接 provider/event feed。 | 推荐方案符合当前用户边界和最小风险；A/B/C 均触发高风险授权。 | 高风险边界；防止 CR154 变成 runtime / data / trading CR。 | 如未来需要真实运行或数据，另起 runtime_authorization / data authorization gate。 |

### CP2 追加字段

| 项 | 内容 |
|---|---|
| 用户真实意图 | 先把三类策略共用生产可靠性 gate 设计成可审计本地 contract，不启动真实 lake、runtime、broker 或真实发布。 |
| 场景覆盖 | UC-58 / UC-59 / UC-60 的横切 blocker：backtest traps、OOS / CV、PIT universe、impact / capacity、regime / attribution / reconciliation 和 admission default policy。 |
| 认知盲区补充 | 真实 TCA、真实 universe、真实 reconciliation、真实 feed/runtime 不是本轮能力；first wave 只能证明 contract semantics。 |
| Scenario Gray Areas | SGQ-CR154-001：独立 CR；SGQ-CR154-002：first-wave 横切 gate；SGQ-CR154-003：no-runtime/no-real-data；SGQ-CR154-004：FU-CR152-001 不占锁；SGQ-CR154-005：CR151/152/153 relation。 |
| Deferred Ideas | real TCA, real universe data, real lake/NAS/provider, real runtime/broker reconciliation, live event feed/listener, registry/store/catalog writes, test taxonomy/provenance hygiene。 |
| 用户选择影响 | approve 后进入 CP3 设计，并要求 CP3 落实 `CP3-DC-CR154-001`；不会启动实现或真实外部操作。 |
| 回退方式 | 修改范围后重写 CP2；reject 则 CR154 保持 cp2_pending / cancelled / candidate，不进入 CP3。 |
| discussion log / checkpoint | `process/discussions/CP2-CR154-SCENARIO-DISCUSSION-LOG.md`；`process/checks/CP2-CR154-DISCUSSION-CHECKPOINT.json`。 |

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 7 |
| 必须用户决策 | `DQ-CP2-CR154-BACKTEST-TRAP-GATE`、`DQ-CP2-CR154-CV-GOVERNANCE`、`DQ-CP2-CR154-SURVIVORSHIP-UNIVERSE-GATE`、`DQ-CP2-CR154-CAPACITY-IMPACT-GATE`、`DQ-CP2-CR154-REGIME-ATTRIBUTION-RECONCILIATION-SLOTS`、`DQ-CP2-CR154-ADMISSION-GATE-DEFAULT-POLICY`、`DQ-CP2-CR154-NO-RUNTIME-NO-REAL-DATA-BOUNDARY` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 表示接受上表 7 项推荐方案。 |
| 不表示授权 | 不表示授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/event store/model registry/live listener/real event feed/real order/real data validation/real reconciliation/true release execution。 |
| 修改: <具体修改点> | 用户可回复具体修改范围、first wave、gate relation、default policy 或授权边界。 |
| reject | 用户可拒绝本 CP2，CR154 将不进入 CP3。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-02T23:58:00+08:00
- 修改意见：接受推荐方案；明确 `DQ-CP2-CR154-BACKTEST-TRAP-GATE` 必须携带 `CP3-DC-CR154-001`，在 CP3 拆出 multiple-testing、FDR/BH、WRC/SPA、PBO/CSCV、DSR/deflation、trial count 等可审计证据位，避免只做标签枚举。
- 风险接受项：接受 CR154 first wave 继续保持 local/static/fixture-only；不授权真实 runtime、真实数据、broker、feed、reconciliation 或 publish。
- 已接受决策项：`DQ-CP2-CR154-BACKTEST-TRAP-GATE`、`DQ-CP2-CR154-CV-GOVERNANCE`、`DQ-CP2-CR154-SURVIVORSHIP-UNIVERSE-GATE`、`DQ-CP2-CR154-CAPACITY-IMPACT-GATE`、`DQ-CP2-CR154-REGIME-ATTRIBUTION-RECONCILIATION-SLOTS`、`DQ-CP2-CR154-ADMISSION-GATE-DEFAULT-POLICY`、`DQ-CP2-CR154-NO-RUNTIME-NO-REAL-DATA-BOUNDARY`
