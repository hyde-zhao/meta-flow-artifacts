---
status: "approved"
version: "0.2"
change: "CR-153"
title: "ADR: Event-Driven Strategy E2E Framework"
created_at: "2026-07-02T17:20:00+08:00"
owner: "host-orchestrator"
approved_by: "user"
approved_at: "2026-07-02T18:00:00+08:00"
---

# ADR: Event-Driven Strategy E2E Framework

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-02 | host-orchestrator | 新增 CR153 CP3 ADR：event-specific gate + adapter、event metadata no-store boundary、extend existing anchors、fixture-only validation、CR154 dependency。 |
| 0.2 | 2026-07-02 | host-orchestrator | 按 CP3 评审同步 HLD v0.2：EV-GAP 编号保持 E2E review 原义，EV-GAP-7 多重检验 / 数据窥探进入 first-wave slot，算法仍 deferred。 |

## ADR-CR153-001: Event-Specific Admission Gate with Shared Status Adapter

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | CR151 covers multifactor statistical admission; CR152 covers ML admission. Event-driven strategies need event time, event study method and event-to-trace evidence that does not fit either schema. |
| Decision | Define an event-specific admission gate summary and adapter that maps to `PASS / FAIL / NEEDS_REVIEW / BLOCKED` and `StrategyAdmissionPackage` linkage. |
| Alternatives | A. Reuse CR151 gate directly; B. define fully separate event status model. |
| Pros | Keeps event evidence clear while preserving shared governance semantics. |
| Cons | Adds one event companion gate and adapter contract. |
| Impact | CP4/CP5 must include event gate contract and adapter tests. |
| Rollback / Switch | If CP5 proves event evidence can be represented without loss in an existing object, adapter may become a thin helper, but four-state semantics must remain. |

## ADR-CR153-002: Event Metadata Contract Is Not Event Store / Catalog / Registry Write

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | CR153 needs event identity, timestamps, method refs and trace refs. These are metadata contracts, not storage infrastructure. |
| Decision | CR153 may define metadata-only fields and validators, but must not create event store writers, catalog pointer mutations, registry writes, provider subscriptions or live listeners. |
| Alternatives | A. Add local event store writer; B. integrate provider feed/listener; C. update catalog current truth. |
| Pros | Preserves static/fixture-only authorization and keeps validation side-effect free. |
| Cons | Cannot claim event store, feed or production current truth readiness. |
| Impact | CP5/CP6 must treat verbs like listen/subscribe/fetch/write/publish/register/upload/sync/mutate/set_current as out of scope for event/store/catalog state. |
| Rollback / Switch | Store, catalog or runtime operations require a later formal CR and authorization gate. |

## ADR-CR153-003: Extend Existing Research / Backtest / Admission Anchors

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | Existing `ResearchDatasetSpec`, `BacktestRunSpec` and `StrategyAdmissionPackage` already carry adjacent research/backtest/admission semantics. |
| Decision | CR153 first wave must extend or compose existing anchors. Narrow event companion objects are allowed only when they reference anchor objects and do not replace them. |
| Alternatives | A. New parallel event framework; B. documentation-only evidence. |
| Pros | Avoids duplicate strategy frameworks and keeps UC-60 linked to existing admission chain. |
| Cons | Requires explicit contract delta mapping in Story design. |
| Impact | CP4/CP5 must include an anchor delta table for every Story that adds event contracts. |
| Rollback / Switch | If a source anchor is defective, fix or wrap the anchor; do not fork semantics silently. |

## ADR-CR153-004: Deterministic Fixture-Only Validation Boundary

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | CP2 approved local/static/fixture-only scope. Real event feeds and live listeners would require provider/runtime/credential authorization. |
| Decision | Use deterministic local fixtures with event time ordering, PIT/revision, event study method, test family and trace positive/negative cases. All forbidden operation counters must remain zero. |
| Alternatives | A. Tiny schema-only fixtures; B. real provider feed validation. |
| Pros | Provides meaningful static verification while respecting authorization. |
| Cons | Does not prove real feed quality, real alpha, production readiness or trading readiness. |
| Impact | CP5 test plan must define fixture fields, positive cases, negative cases and forbidden operation counters. |
| Rollback / Switch | Real data validation requires a later data/runtime authorization gate. |

## ADR-CR153-005: CR154 Owns Full Cross-Strategy Reliability Governance

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | Event CV, survivorship-free universe, capacity/impact, market impact, regime and reconciliation are necessary for production-grade reliability but exceed CR153 first wave. |
| Decision | CR153 keeps machine-visible slots/status/refs/n/a reasons for these risks; full governance is deferred to CR154 or later formal CRs. |
| Alternatives | A. Expand CR153 into full production reliability scope; B. omit these risks entirely. |
| Pros | Controls scope while preserving audit visibility for known blockers. |
| Cons | CR153 cannot claim full UC-60 production E2E readiness. |
| Impact | CP7/CP8 must state residual risks and CR154 dependency. |
| Rollback / Switch | If the user requires full governance before event framework work, reopen scope through CP2 or create a new CR. |

## Not Authorized

- source implementation before later CP5 approval
- Story decomposition before CP3 approval
- real event feed or live listener
- real lake read/write
- NAS read/write/sync/restore/chmod/chgrp/metadata normalization
- provider fetch
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/paper/live/trading runtime
- broker read/write/submit/cancel/account query
- credential or `.env` read
- external framework clone/install/run
- Git remote write
- catalog pointer mutation
- feature store, label store, event store, model registry or prediction store writes
- real order flow or real data validation

## Downstream Obligations

| Stage | Obligation |
|---|---|
| CP4 | Split Stories only after CP3 approval; preserve candidate groups or document changes. |
| CP5 | Each design evidence item must include anchor mapping, fixture cases, no-write/no-runtime guardrails, original EV-GAP traceability and design-delta obligations. |
| CP5 | Event study / admission design must include `multiple_testing_or_data_snooping_slot` with family id, tested window count, correction method, adjusted p-value, status, report ref or n/a reason; White/Hansen/PBO/DSR algorithms remain out of CR153 first wave unless separately authorized. |
| CP6 | Implementation must use local/static/fixture tests only; any feed/listener/store/registry/runtime writer is a scope violation. |
| CP7 | Verification wording must state fixture semantics only, not real event feed, real returns or trading readiness. |
| CP8 | Release readiness must carry CR154 dependency and no-runtime/no-store limitations. |
