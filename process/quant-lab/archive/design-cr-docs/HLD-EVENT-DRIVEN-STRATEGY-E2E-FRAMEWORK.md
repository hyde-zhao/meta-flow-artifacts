---
status: "approved"
version: "0.2"
change: "CR-153"
title: "Event-Driven Strategy E2E Framework HLD"
created_at: "2026-07-02T17:20:00+08:00"
owner: "host-orchestrator"
scope_boundary: "local/static/fixture-only; event metadata contracts only; no event store/catalog/registry write, no real feed, no live listener, no runtime or trading"
approved_by: "user"
approved_at: "2026-07-02T18:00:00+08:00"
---

# HLD: Event-Driven Strategy E2E Framework

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-02 | host-orchestrator | CR153 CP3 初稿：定义事件驱动策略 first-wave 架构、事件时间语义、event study method slots、event-specific admission gate、existing-anchor extension、fixture-only 边界和 CR154 split。 |
| 0.2 | 2026-07-02 | host-orchestrator | 按 CP3 评审修正 §11：保留 E2E review 原始 EV-GAP-1..9 编号；新增 multiple_testing_or_data_snooping_slot，明确 White/Hansen/PBO/DSR 等算法仍为 slot-only / deferred。 |

## 1. Problem

UC-60 需要从事件研究、事件信号、target/order-intent trace 到策略准入形成可审计链路。当前 CR151 多因子统计 gate 和 CR152 ML gate 已建立共享 admission 状态基础，但事件驱动策略仍缺三类基础能力：

| Gap | Risk |
|---|---|
| Event time semantics | `event_occurred_at`、`event_announced_at`、`event_available_at` 与 `decision_time` 混用会造成 look-ahead。 |
| Event study method contract | 没有 estimation window、event window、normal return model、CAR/BHAR/calendar-time、test family、cluster/endogeneity 等结构化证据。 |
| Event-specific admission and trace | 事件研究 PASS/FAIL 无法映射到 strategy admission；event->signal->target/order-intent 边界容易被误读成 runtime/order flow。 |

CR153 CP3 的目标是确认一个可进入 CP4/CP5 的架构。它不实现真实事件 feed、不启动 listener、不接 QMT/broker、不写 event store/catalog/model registry，也不声明生产或交易 readiness。

## 2. Goals and Success Criteria

| ID | Goal | Measurable Success Criteria |
|---|---|---|
| G1 | 收敛事件 first-wave 架构 | HLD 覆盖 9 个 CP2 accepted DQ，并把每项映射到 module boundary、contract delta 或 deferred slot。 |
| G2 | 复用现有契约 | 至少 5 个现有锚点被列入 extension anchors：`ResearchDatasetSpec`、`BacktestRunSpec`、`StrategyAdmissionPackage`、`StrategyAdmissionStatisticalGate`、`MLAdmissionGate`。 |
| G3 | 固化 event metadata no-store 边界 | HLD/ADR/CP3 checkpoint 至少 3 处明确 event store/catalog/registry write 不授权。 |
| G4 | 支持 CP5 设计证据 | CP3 输出 5 个 CP3 DQ、5 个 ADR、1 个 contract delta table、1 个 failure-path table 和 candidate Story groups。 |
| G5 | Fixture-only 可验证 | Fixture contract 至少包含 12 个概念：event id、entity id、event type、occurred_at、announced_at、available_at、decision_time、estimation window、event window、normal return model、test family、forbidden operation counters。 |

## 3. Non-Goals

| Non-goal | Boundary |
|---|---|
| Real event feed or live listener | 不运行实时监听、订阅、轮询或 provider fetch。 |
| Event store / catalog / registry | 不创建、更新、发布或 mutate event store、catalog pointer、model registry 或任何 production current truth。 |
| Runtime / order execution | 不启动 QMT、MiniQMT、xtquant、gateway、simulation、paper、live、broker 或真实 order flow。 |
| Full event study statistics library | Patell、BMP、bootstrap、cluster robust variance、PSM/IV/matching 等只作为 method slot/status/ref/n/a reason，不实现算法。 |
| Full CV / survivorship / capacity governance | CR153 只提供 machine-visible slots；完整治理交给 CR154 或后续 CR。 |
| Real data validation | 不读真实 lake、NAS、provider、credential、catalog current truth 或生产数据快照。 |

## 4. Architecture Gray Areas

**讨论日志**：`process/discussions/CP3-CR153-HLD-DISCUSSION-LOG.md`

**讨论恢复点**：`process/checks/CP3-CR153-DISCUSSION-CHECKPOINT.json`

| AGA ID | Key Question | Recommendation | Status |
|---|---|---|---|
| AGA-CR153-01 | Event gate 独立还是复用 CR151/CR152 gate？ | Event-specific gate + shared four-state adapter。 | decision-item |
| AGA-CR153-02 | Event metadata 是否等于 event store/catalog/registry？ | Metadata-only; no store/catalog/registry write。 | decision-item |
| AGA-CR153-03 | 是否重建 event framework？ | Extend/compose existing anchors。 | decision-item |
| AGA-CR153-04 | 是否用真实 feed/listener 证明？ | Deterministic local fixtures only。 | decision-item |
| AGA-CR153-05 | 完整 CV/survivorship/capacity 是否进入 CR153？ | Slots only; full governance deferred to CR154。 | decision-item |

## 5. Candidate Architectures

| Option | Description | Pros | Cons | Decision |
|---|---|---|---|---|
| A. Contract extension + Event-specific gate + adapter | 在既有研究/回测/admission 契约上增加 event companion objects，event gate 独立表达事件证据，通过 adapter 复用 CR151/CR152 四态语义。 | 语义清晰、可 fixture-only 测试、与 admission 链路收敛。 | 需要新增 event companion contracts 和 adapter。 | Recommended |
| B. Reuse CR151 statistical gate directly | 把事件研究证据压进 `StrategyAdmissionStatisticalGate` refs/reasons。 | 新模块少。 | 事件窗口、test family、聚类/内生性等语义被压扁，污染多因子 gate 职责。 | Rejected |
| C. Parallel event framework | 新建完整 event strategy contract family，不复用 existing anchors。 | 命名独立。 | 形成平行 strategy framework，重复 research/backtest/admission contract。 | Rejected |
| D. Runtime event pipeline | 建 listener/feed/order-intent runtime chain。 | 接近完整 UC-60 生产链路。 | 超出 CP2 授权，涉及 provider/runtime/broker/credential。 | Rejected / Out of scope |

## 6. Recommended Architecture

CR153 采用 Option A：metadata-only event contract extensions + event-specific admission gate + shared status adapter。

```text
Local deterministic event fixture
        |
        v
ResearchDatasetSpec extension anchor
        |
        v
EventResearchSpec
  - event_occurred_at
  - event_announced_at
  - event_available_at
  - decision_time
        |
        v
EventRevisionPITGate ----> EventStudyMethodSpec
        |                    |
        v                    v
EventStudyTestReport   overlap / cluster / endogeneity / CV / universe slots
        \                    /
         \                  /
          v                v
          EventStrategyAdmissionGate
                    |
                    v
 CR151/CR152 four-state adapter + StrategyAdmissionPackage linkage
                    |
                    v
 event -> signal -> target/order-intent trace metadata
```

## 7. Contract Delta Table

| Existing Anchor | Current Role | CR153 Extension / Companion | Boundary |
|---|---|---|---|
| `ResearchDatasetSpec` in `engine/research_production_contracts.py` | Research dataset metadata, date window, feature/label refs, source-of-truth guards. | Add/compose `EventResearchSpec` with event identity, event type, entity keys, three-time semantics, decision time and source snapshot refs. | No lake/provider read; no event store write. |
| `FeatureAvailabilityContract` / time fields in `engine/research_production_contracts.py` | Decision-time and availability-time leakage guards. | Add event-specific `EventTimeSemantics` and `EventRevisionPITGate`; `available_at > decision_time` must fail closed. | No automatic inference from occurred/announced time. |
| `BacktestRunSpec` in `engine/backtest_production_contracts.py` | Backtest metadata, signal set, portfolio policy, cost/slippage refs. | Add/compose event study refs, estimation/event window metadata and event-to-signal trace refs. | No real backtest execution or runtime simulation. |
| `StrategyAdmissionStatisticalGate` in `engine/strategy_admission_statistical_gate.py` | Shared four-state statistical gate semantics. | Reuse status semantics through adapter; do not reuse CR151 report schema as event report schema. | Adapter maps status only. |
| `MLAdmissionGate` in `engine/ml_strategy_admission_gate.py` | CR152 first-wave ML admission gate status semantics. | Reuse adapter pattern for evidence-specific gate plus shared admission language. | No ML evidence mixing. |
| `StrategyAdmissionPackage` in `engine/strategy_admission_package.py` | Offline admission evidence package with runtime claim blockers. | Add/compose event gate summary fields and event-to-signal/order-intent trace metadata refs. | Does not imply paper/live/broker readiness. |

## 8. Module Responsibilities

| Module / Area | Responsibility | Inputs | Outputs | Failure Path |
|---|---|---|---|---|
| `engine.research_production_contracts` | Own event research companion contracts near `ResearchDatasetSpec`. | Static event fixture metadata and dataset refs. | `EventResearchSpec`, `EventTimeSemantics`, validation issues. | Missing three-time semantics or mutable refs -> `BLOCKED`. |
| Event PIT companion module | Evaluate revision/PIT availability evidence. | `EventResearchSpec`, revision refs, decision time. | `EventRevisionPITGate`. | `available_at > decision_time` -> `BLOCKED`; missing revision policy -> `NEEDS_REVIEW` or `BLOCKED` per CP5. |
| Event study method companion module | Express estimation window, event window, normal return model, CAR/BHAR/calendar-time and test family slots. | Event fixture, backtest refs, method config. | `EventStudyMethodSpec`, `EventStudyTestReport`. | Missing mandatory method evidence -> `BLOCKED`; unsupported method active -> `NEEDS_REVIEW` or `BLOCKED`. |
| Event admission companion module | Evaluate event first-wave evidence into event gate summary. | PIT gate, method report, multiple-testing/data-snooping slot, cluster/endogeneity/CV/universe slots, operation counters. | `EventStrategyAdmissionGate`, issues and status. | Mandatory evidence missing -> `BLOCKED`; semantic failure -> `FAIL`; incomplete reviewable evidence -> `NEEDS_REVIEW`. |
| Admission adapter | Map event gate to shared four-state semantics and package linkage. | `EventStrategyAdmissionGate`. | Package summary fields and blocked reasons. | Unknown status or adapter mismatch -> `BLOCKED`. |
| Trace contract | Preserve event->signal->target/order-intent metadata boundary. | Event gate summary, signal refs, order-intent metadata refs. | Trace refs for `StrategyAdmissionPackage`. | Any real order or broker operation counter nonzero -> `BLOCKED`. |

## 9. Event Metadata vs Store / Runtime Boundary

| In Scope Metadata Contract | Explicitly Out of Scope Store / Runtime Operation |
|---|---|
| `event_id`, `event_type`, `entity_id`, `event_occurred_at`, `event_announced_at`, `event_available_at`, `decision_time`, `revision_policy`, `source_snapshot_ref`, `method_ref`, `test_report_ref`, `trace_ref`, `limitations`, `operation_counts`. | Creating event store rows, updating catalog pointers, publishing registry entries, subscribing provider feeds, running listeners, mutating production current truth, sending broker orders. |
| Static validation that required fields exist, times are ordered and forbidden counters are zero. | Any API, filesystem or network operation that writes event/store/catalog/registry state or interacts with runtime/trading systems. |
| Fixture claims limited to contract semantics. | Claims of real event feed coverage, production readiness, paper/live readiness or trading readiness. |

CP5/CP6 guardrail: any function whose primary action is `listen`, `subscribe`, `fetch`, `write`, `publish`, `register`, `promote`, `upload`, `sync`, `persist`, `mutate`, `set_current`, `submit_order`, `cancel_order` or `query_account` is outside CR153 first wave.

## 10. Key Flows

### Flow 1: Fixture Contract Assembly

1. Build deterministic event rows with event id, entity id, event type, occurred/announced/available/decision times and source snapshot refs.
2. Build `ResearchDatasetSpec`-anchored `EventResearchSpec`.
3. Build `EventRevisionPITGate` to fail closed when event availability is after decision time or revision policy is missing.
4. Build `EventStudyMethodSpec` with estimation window, event window, normal return model, CAR/BHAR/calendar-time slots and test family slots.
5. Build `multiple_testing_or_data_snooping_slot` with family id, tested window count, correction method, adjusted p-value, status, report ref or n/a reason.
6. Build overlap/cluster/endogeneity, event CV and universe PIT audit slots.
7. Evaluate `EventStrategyAdmissionGate` and adapter summary.
8. Link event gate and event->signal->target/order-intent trace metadata to `StrategyAdmissionPackage`.

### Flow 2: Fail-Closed Evidence

| Failure | Expected Status | Reason |
|---|---|---|
| Missing `event_available_at` | `BLOCKED` | Event availability cannot be proven. |
| `event_available_at > decision_time` | `BLOCKED` | Look-ahead risk. |
| Missing estimation window or event window | `BLOCKED` | Event study method contract incomplete. |
| Active method requires unsupported Patell/BMP/bootstrap implementation | `NEEDS_REVIEW` or `BLOCKED` | First wave is method slot-only; CP5 must set exact enforcement. |
| Test family evidence absent | `BLOCKED` | Ordinary forward return is insufficient for UC-60. |
| Multiple testing / data snooping slot absent | `BLOCKED` | E2E review EV-GAP-7 is a blocker; first wave must preserve correction evidence even when algorithms are not implemented. |
| Cluster/endogeneity evidence slot absent | `NEEDS_REVIEW` or `BLOCKED` | Risk must remain machine visible. |
| Event-to-order trace includes real broker/order operation | `BLOCKED` | Runtime/trading not authorized. |
| Event store/catalog/registry write counter nonzero | `BLOCKED` | Store/write boundary violated. |

## 11. Use Case and Gap Traceability

| Use Case / Gap | Original Review Meaning | Architecture Coverage | Verification Mode |
|---|---|---|---|
| UC-60 event-driven strategy E2E | Full event-driven research-to-admission-to-trace journey. | First-wave metadata chain from event research to event admission and trace. | Unit tests + fixture validation + CP7 evidence after later gates. |
| CP2-DQ event time semantics | CP2-approved first-wave requirement, not an EV-GAP renumbering. | `EventTimeSemantics` and `EventRevisionPITGate`; `event_available_at > decision_time` fails closed. | Deterministic PIT/leakage fixtures. |
| EV-GAP-1 | Estimation window + normal return benchmark model not explicitly selected. | `EventStudyMethodSpec` requires estimation window, event window and normal return model fields. | Static contract validation and missing-method fail-closed fixtures. |
| EV-GAP-2 | Significance test family unspecified. | `EventStudyTestReport` slots for Patell / BMP / generalized sign / rank / bootstrap with status, refs, sample count and adjusted p-value fields. | Slot presence and fail-closed fixtures; algorithms deferred. |
| EV-GAP-3 | Event endogeneity / self-selection has no treatment mechanism. | Structured `endogeneity_treatment_slot` with status, refs and n/a reason. | Slot validation only in first wave. |
| EV-GAP-4 | Overlap / clustering event windows have no variance adjustment. | Structured overlap / cluster report slots with status, refs and n/a reason. | Slot validation only in first wave. |
| EV-GAP-5 | Long-horizon BHAR default bias; CAR/calendar-time choice not specified. | `EventStudyMethodSpec` includes CAR, BHAR and calendar-time method slots plus support status. | Slot validation; full long-horizon statistics deferred. |
| EV-GAP-6 | PIT / event revision look-ahead gate incomplete. | `EventRevisionPITGate` and event availability/revision refs. | Availability/revision fixtures. |
| EV-GAP-7 | Multiple testing / data snooping correction absent. | `multiple_testing_or_data_snooping_slot` in `EventStudyTestReport` or `EventStrategyAdmissionGate` with `family_id`, `tested_window_count`, `correction_method`, `adjusted_p_value`, `status`, and `report_ref` or `n/a_reason`. | Slot presence and fail-closed fixtures; White Reality Check / Hansen SPA / Romano-Wolf / Bonferroni / FDR / PBO / DSR algorithms deferred. |
| EV-GAP-8 | Walk-forward / OOS / purged-embargoed CV is not a first-class stage. | Event CV slot and split audit refs. | Slot validation; full CV deferred to CR154. |
| EV-GAP-9 | Survivorship bias / universe reconstruction lacks explicit gate. | Universe PIT audit slot. | Slot validation; full survivorship-free universe gate deferred to CR154. |
| EV-GAP-17 | Event-to-order trace lacks full state machine / compliance report. | Trace metadata from event to signal/target/order-intent only; no broker/runtime counters and blocked claims. | Static trace contract only; full runtime/compliance state machine deferred. |
| CP3-DQ gate relation | CP3 architecture decision, not an EV-GAP renumbering. | Event-specific gate plus shared CR151/CR152 adapter. | Adapter mapping tests after later gates. |

## 12. Fixture Data Contract

| Requirement | Minimum |
|---|---|
| Event identity | `event_id`, `event_type`, stable entity id and source snapshot ref. |
| Required timestamps | `event_occurred_at`, `event_announced_at`, `event_available_at`, `decision_time`. |
| Revision/PIT | revision policy id, revision source ref, PIT status and n/a reason where unavailable. |
| Method fields | estimation window start/end, event window start/end, normal return model, return horizon. |
| Test family fields | test family id, status, sample count, raw p-value, adjusted p-value, report ref or n/a reason. |
| Multiple testing / data snooping fields | `family_id`, `tested_window_count`, `correction_method`, `adjusted_p_value`, `status`, and `report_ref` or `n/a_reason`; algorithms remain slot-only in CR153 first wave. |
| Bias risk slots | overlap, cluster, endogeneity, CV split audit and universe PIT audit slots. |
| Trace fields | signal ref, target/order-intent metadata ref, no broker/runtime operation counters. |
| Positive fixture | At least 1 passing event chain with ordered times and complete slots. |
| Negative fixture | At least 1 look-ahead event where available_at exceeds decision_time. |
| Operation counters | All forbidden operation counters must be zero. |
| Claim limit | Fixture proves contract semantics only; it does not prove event alpha, feed reliability or trading readiness. |

## 13. Security and Authorization Boundary

CR153 CP3 approval does not authorize:

- source implementation before later CP5 approval
- Story decomposition before CP3 approval
- real lake read/write
- NAS read/write/sync/restore/chmod/chgrp/metadata normalization
- provider fetch or live event feed
- live event listener
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/paper/live/trading runtime
- broker read/write/submit/cancel/account query
- credential or `.env` read
- external framework clone/install/run
- Git remote write
- catalog pointer mutation
- feature store, label store, event store, model registry or prediction store writes
- real order flow or real data validation

## 14. Risks and Mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| Event metadata contract is misread as event store readiness | high | HLD/ADR/checkpoint state no-store boundary; CP5 requires forbidden operation counters. |
| Fixture PASS is misread as real event feed or trading readiness | high | Admission package limitations and CP7/CP8 wording must say static contract semantics only. |
| Event method slots are mistaken for implemented Patell/BMP/bootstrap/statistics | medium | Method slot must expose support state, refs and n/a reason; active unsupported method fails closed or needs review. |
| CR153 duplicates existing strategy framework anchors | high | CP5 must include contract delta mapping; no new object may replace `ResearchDatasetSpec`, `BacktestRunSpec` or `StrategyAdmissionPackage`. |
| CR154 deferred risks become invisible | medium | CV, survivorship, capacity/impact/regime/reconciliation must remain machine-visible slots or deferred refs. |
| EV-GAP traceability drifts from original E2E review numbering | medium | §11 preserves original EV-GAP-1..9 meanings and uses CP2-DQ / CP3-DQ rows for additional architecture decisions. |

## 15. Tentative Story Groups After CP3 Approval

Story decomposition is not authorized until CP3 is approved. If approved, CP4 may split along these candidate groups:

| Candidate Story | Scope | Design Evidence Level |
|---|---|---|
| CR153-S01 | EventResearchSpec, three-time semantics and EventRevisionPITGate. | full-lld |
| CR153-S02 | EventStudyMethodSpec, estimation/event window and method/test family slots. | full-lld |
| CR153-S03 | Overlap/cluster/endogeneity, event CV and universe PIT audit slots. | full-lld |
| CR153-S04 | EventStrategyAdmissionGate and CR151/CR152 shared status adapter. | full-lld |
| CR153-S05 | Event-to-signal/target/order-intent trace contract and no-runtime evidence wiring. | technical-note |

## 16. Gotchas

1. Do not infer `event_available_at` from `event_occurred_at` or `event_announced_at`; missing availability is a gate issue.
2. Do not treat event metadata refs as event store rows or catalog current truth.
3. Do not use CR151 statistical gate schema as the event report schema; reuse only four-state semantics through adapter.
4. Do not let event-to-order trace create or submit any real order.
5. Do not claim CR153 covers full CV, survivorship-free universe, capacity, impact, regime or reconciliation governance.

## 17. Open Questions

| ID | Question | Status | Resolution |
|---|---|---|---|
| O-CR153-CP3-001 | Should CR153 include live event listener or real feed? | RESOLVED | No. CP2 approved local/static/fixture-only scope. |
| O-CR153-CP3-002 | Should event contracts rebuild a separate strategy framework? | RESOLVED | No. Extend/compose existing anchors. |
| O-CR153-CP3-003 | Should method slots implement full Patell/BMP/bootstrap/cluster robust/PSM/IV algorithms? | RESOLVED | No. CR153 first wave is slot/status/ref/n/a reason only. |
| O-CR153-CP3-004 | Should CV/survivorship/capacity/regime/reconciliation be complete in CR153? | RESOLVED | No. Keep slots and defer full governance to CR154. |
