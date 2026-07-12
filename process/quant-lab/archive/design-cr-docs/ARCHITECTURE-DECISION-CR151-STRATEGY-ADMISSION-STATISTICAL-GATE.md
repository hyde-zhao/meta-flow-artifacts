---
status: "draft-for-cp3"
version: "1.1"
cr_id: "CR-151"
title: "ADR: CR151 Strategy Admission Statistical Gate"
source_hld: "process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-01T23:24:00+08:00"
authorization_boundary: "design-only/static-only; no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote"
---

# ADR: CR151 Strategy Admission Statistical Gate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-01 | host-orchestrator | 初版：记录 CR151 CP3 的 module boundary、Wave boundary、status model 和 validation boundary。 |
| 1.1 | 2026-07-01 | host-orchestrator | 明确 Wave A 不覆盖 MF-GAP-2 扩展评价统计、MF-GAP-4 regime 分层和 MF-GAP-7 因子相关性聚类 / 去重；上述项 deferred to CR154 / follow-up。 |

## ADR-CR151-001: Dedicated Statistical Gate Contract Module

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Create a dedicated metadata-only statistical gate contract module, tentatively `engine.strategy_admission_statistical_gate`. |
| Alternatives | A. embed directly in `engine.mature_multifactor_research`; B. put all logic into `engine.strategy_admission_package`; C. documentation-only. |
| Pros | Isolated tests, clearer module ownership, future ML/event reuse, lower risk of runtime claim confusion. |
| Cons | Adds one module and one linkage story. |
| Impact | Source module boundary, tests, CR151 Story split, CR152/CR153 reuse option. |
| Rollback / Switch | If CP5 proves the module has no cross-strategy value and only one trivial helper remains, merge the helper into mature multifactor linkage before implementation. |

## ADR-CR151-002: Wave A Only For CR151

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | scope |
| Recommendation | Implement Wave A only: multiple testing/FDR, robust factor statistics, walk-forward/OOS, PBO/DSR, and the aggregate statistical gate. |
| Alternatives | A. include Wave B capacity/impact, benchmark-relative, PIT audit, extended factor diagnostics, regime-aware validation and factor correlation clustering now; B. documentation-only; C. only PBO/DSR. |
| Pros | Keeps CP5 and implementation tractable; directly addresses E2E review blockers C1/C2 and MF-GAP-1/2/4. |
| Cons | Leaves capacity, benchmark-relative reporting, PIT universe audit, MF-GAP-2 extended evaluation diagnostics, MF-GAP-4 regime-aware validation and MF-GAP-7 factor correlation clustering / de-duplication for future CR154 or follow-up. |
| Impact | Story count, tests, release wording and roadmap follow-up. |
| Rollback / Switch | If user changes route to production-grade turnover before CR152/153, reopen scope and split CR154 earlier. |

## ADR-CR151-003: Four-State Fail-Closed Gate Model

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Use `PASS`, `FAIL`, `NEEDS_REVIEW`, and `BLOCKED` as the statistical gate statuses. |
| Alternatives | A. binary pass/fail; B. always blocked until real data; C. reuse existing admission enum without added semantics. |
| Pros | Distinguishes missing evidence from threshold failure and borderline evidence; supports fail-closed mandatory evidence. |
| Cons | Adds one more state than binary admission. |
| Impact | Test matrix, CP7 verification, CP8 release wording, blocked-claim mapping. |
| Rollback / Switch | If existing `AdmissionStatus` has an equivalent typed status by CP5, adapter may map the four states to the existing enum without losing semantics. |

## ADR-CR151-004: Static/Fixture-Only Validation Boundary

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | security |
| Recommendation | Keep CR151 validation static/fixture-only; no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote operation is authorized. |
| Alternatives | A. authorize real lake read; B. authorize external framework run; C. authorize NAS/report write. |
| Pros | Matches CP2 approval, avoids runtime risk, keeps implementation locally verifiable. |
| Cons | Cannot claim real strategy performance or production turnover. |
| Impact | CP6/CP7 evidence, CP8 release decision, no-real-operation tests. |
| Rollback / Switch | Any real data/runtime validation requires separate runtime authorization gate or formal CR. |

## Consistency Notes

| Topic | Decision |
|---|---|
| ResearchDatasetSpec / BacktestRunSpec | Existing source contracts are present and are extension anchors; do not recreate them. |
| StrategyAdmissionPackage | May consume statistical gate refs/blocked reasons, but must not convert statistical PASS into runtime readiness. |
| CR150 completion map | CR151 may extend linkage with statistical gate refs, but CR150 static-only release remains historically unchanged. |
| Wave B | Deferred to follow-up / CR154 unless CP3 is explicitly modified. This includes capacity/impact, benchmark-relative IR/TE/Active Share, PIT universe audit, MF-GAP-2 extended evaluation diagnostics, MF-GAP-4 regime-aware validation, and MF-GAP-7 factor correlation clustering / de-duplication. |
| MF-GAP-2 boundary | CR151 Wave A covers Newey-West or equivalent robust t/p-value over IC / RankIC-style evidence. IC decay, half-life, turnover, liquidity/capacity view, orthogonalization, monotonicity and quantile spread are not mandatory Wave A fields. |
| MF-GAP-4 boundary | CR151 Wave A covers walk-forward / OOS folds only; regime-aware validation is explicitly deferred. |
| MF-GAP-7 boundary | Factor correlation clustering and redundancy de-duplication are not covered by CR151 Wave A and remain follow-up / CR154 scope. |

## Not Authorized

- real lake read or write
- NAS read / write / sync / chmod / chgrp / metadata normalization
- provider fetch
- QMT / MiniQMT / xtquant runtime
- simulation / paper / live / trading runtime
- broker read / write / submit / cancel / real account query
- credential or `.env` read
- external framework clone / install / run
- Git remote write
- catalog pointer mutation
