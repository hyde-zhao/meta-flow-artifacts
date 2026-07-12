---
status: "approved"
version: "1.1"
cr_id: "CR-157"
title: "ADR: Stage 2 Multifactor Research Framework Upgrade"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-05T12:48:20+08:00"
authorization_boundary: "design-only/static-fixture-only; no implementation, real lake write, NAS/provider/credential/QMT/runtime/simulation/live/trading/broker/external framework/Git remote/catalog/store/registry operation"
---

# ADR: CR157 Stage 2 Multifactor Research Framework Upgrade

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-05 | host-orchestrator | 初版：记录 CR157 CP3 的 builder、evidence、handoff、deferred adapter 和 no-runtime 决策。 |
| 1.1 | 2026-07-05 | host-orchestrator | CP3 人工确认通过：5 项 ADR 推荐方案被用户 approve 接受。 |

## ADR-CR157-001: Contract-First Stage 2 Exit Hardening

| Field | Decision |
|---|---|
| Status | accepted |
| Type | architecture |
| Recommendation | Harden the existing Stage 2 mature multifactor framework contracts instead of creating a new parallel framework. |
| Alternatives | A. documentation-only; B. broad cross-strategy adapter platform; C. rewrite Stage 2 package schemas. |
| Pros | Reuses current code, keeps schema continuity, closes the Stage 2 exit, and keeps CP4 Story split focused. |
| Cons | Requires careful compatibility checks around existing schema names and historical CR030/CR039 refs. |
| Impact | `engine/mature_multifactor_framework.py`, fixture tests, CP4 Story plan, CP8 release wording. |
| Rollback / Switch | If CP5 finds the existing module too broad, split helper functions into a small companion module without changing public dataclass payloads. |

## ADR-CR157-002: ResearchEvidenceIndex Remains Refs-Only

| Field | Decision |
|---|---|
| Status | accepted |
| Type | architecture |
| Recommendation | Keep research evidence traceability as refs-only: index refs, lineage refs, limitations and typed unavailable entries; do not copy full reports or metrics bodies. |
| Alternatives | A. inline evidence contents; B. let Stage 2 fill missing evidence from lake/provider; C. no evidence index change. |
| Pros | Avoids dual truth sources, preserves Stage 2 no-lake boundary, and makes Stage 3 handoff explicit. |
| Cons | Requires consumers to dereference evidence separately in Stage 3. |
| Impact | Evidence index validators, tests, Stage 3 handoff payload, release wording. |
| Rollback / Switch | If a downstream consumer requires small summaries, add bounded metadata summaries while keeping original evidence refs authoritative. |

## ADR-CR157-003: Stage 3 Handoff Is Fail-Closed

| Field | Decision |
|---|---|
| Status | accepted |
| Type | architecture |
| Recommendation | Stage 3 handoff must enumerate 12 required inputs, 13 required evidence refs, execution boundary and blocked_until conditions; missing real refs block Stage 3 package readiness. |
| Alternatives | A. allow partial Stage 3 package; B. mark Stage 2 bundle as Stage 4-ready; C. run Stage 3 automatically. |
| Pros | Makes Stage 3 work clear and prevents premature runtime/simulation claims. |
| Cons | More verbose package and negative tests. |
| Impact | `Stage3ResearchMachineHandoff`, `Stage3MatureResearchPackage`, CP7 verification, CP8 risk wording. |
| Rollback / Switch | CP5 may refine field names, but cannot remove the fail-closed behavior without returning to CP3. |

## ADR-CR157-004: Event And ML Adapters Deferred

| Field | Decision |
|---|---|
| Status | accepted |
| Type | scope |
| Recommendation | Defer event adapter (`DF-CR157-001`) and ML adapter (`DF-CR157-002`) out of CR157 first slice. |
| Alternatives | A. implement event adapter now; B. implement ML adapter now; C. define generic adapter abstraction now. |
| Pros | Prevents premature FEAT-13 coupling and keeps multifactor Stage 2 exit small enough for CP4/CP5. |
| Cons | Cross-strategy adapter parity remains future work. |
| Impact | CP4 Story split must not include event/ML adapter implementation. |
| Rollback / Switch | If user explicitly expands CP3 scope, reopen CP2/CP3 decision and create new adapter stories or child CR. |

## ADR-CR157-005: No-Runtime Guard Is A First-Class Acceptance Criterion

| Field | Decision |
|---|---|
| Status | accepted |
| Type | security |
| Recommendation | Treat no-runtime/no-write as first-class acceptance: permission counters, forbidden import scan, blocked claims and CP8 wording must all remain aligned. |
| Alternatives | A. document-only safety; B. runtime blacklisting; C. allow read-only runtime smoke. |
| Pros | Machine-verifiable and consistent with Stage 2 boundaries. |
| Cons | Does not prove live readiness or real data availability. |
| Impact | CP5 test plan, CP6 tests, CP7 evidence, CP8 not-authorized list. |
| Rollback / Switch | Any runtime or real data work requires a separate runtime_authorization gate or new CR. |

## Consistency Notes

| Topic | Decision |
|---|---|
| CR150 | FactorSpec/FactorRunSpec and framework completion are referenced, not reopened. |
| CR151 | Statistical gate remains upstream evidence; CR157 does not reimplement robust statistics. |
| CR154 | Reliability gates remain complementary; CR157 does not replace production reliability validation. |
| CR155 | Daily baseline artifact is an example consumer/source, not the only Stage 2 package shape. |
| Stage 2 | No lake write, provider fetch, QMT/runtime, simulation/live/trading or publish. |
| Stage 3 | May fill real data/evidence refs only under separate research-machine authorization. |

## Not Authorized

- source/test implementation before CP5 approval
- real lake read or write
- NAS read/write/sync/chmod/chgrp/metadata normalization
- provider fetch
- QMT / MiniQMT / xtquant / gateway operation
- simulation / paper / live / trading
- broker read/write/submit/cancel/account query
- credential or `.env` read
- external framework clone/install/run
- Git remote write
- catalog pointer, store or registry mutation
- production deployment, publish or true release execution
