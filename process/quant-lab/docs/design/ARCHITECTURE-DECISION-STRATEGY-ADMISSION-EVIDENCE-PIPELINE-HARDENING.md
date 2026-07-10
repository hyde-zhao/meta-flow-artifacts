---
status: "draft-for-cp3"
version: "0.1"
cr_id: "CR-161"
title: "ADR: Strategy Admission Evidence Pipeline Hardening"
source_hld: "docs/design/HLD-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING.md"
complexity: "standard"
confirmed: false
confirmed_by: ""
confirmed_at: ""
authorization_boundary: "design-only; no source/test implementation, credentials, .env, real lake, NAS, provider, runtime, broker, external framework, Git remote write or publish"
---

# ADR: CR161 Strategy Admission Evidence Pipeline Hardening

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-09 | meta-se | Initial CP3 ADR draft covering contract-first evidence availability, typed_unavailable fail-closed semantics, CR151/CR154 integration, CR155 negative regression, follow-up split and authorization boundary. |

## Agent / Skill Composition

| Role / Skill | Use in CR161 CP3 |
|---|---|
| `meta-se-critical` | Owns CP3 architecture review/design output for a cross-strategy evidence contract and major ADR. |
| `hld-designer` | Used for Architecture Gray Areas, candidate comparison, traceability, scenario simulation and HLD structure. |
| `checkpoint-manager` | Used for CP3 result JSON shape and decision/precheck items. |
| `blueprint-design` | Not separately invoked in this CP3-only task; the HLD records blueprint applicability and waiver for standalone blueprint artifacts because no Story/implementation route is authorized. |

## Platform Adaptation Differences

CR161 CP3 is a design-only change. It does not modify Python dependencies, platform installation, Agent files, Skill files, source code, tests, runtime commands, data lake paths, NAS routes, QMT gateway behavior, broker integration, provider access, catalog/store/registry/model/prediction paths, external framework setup, Git remotes or release publishing paths.

## ADR-CR161-001: Contract-First Evidence Availability Overlay

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Use a contract-first evidence availability overlay that defines typed evidence envelopes for the seven CR161 objects and integrates them through CR151/CR154 refs and summaries. |
| Alternatives | A. Compute-first pipeline with research-engine instrumentation and analytics implementation; B. documentation-only checklist; C. parallel CR161 gate family. |
| Pros | Matches CP2 approval; avoids unauthorized implementation/runtime/data operations; makes missing evidence explicit; allows later producers to populate the same contracts. |
| Cons | Does not compute FDR/PBO/DSR, fold-level OOS, real TCA or capacity values in the current slice. |
| Impact | HLD, admission wording, future Feature design triggers, CR151/CR154 integration and CP7 regression scope. |
| Risk | Contract semantics may be misread as computed proof. |
| Rollback / Switch | Switch to compute-first only after explicit user approval to reroute through CP4/CP5/CP6 or a follow-up CR. Keep current CP3 result at human gate until then. |

## ADR-CR161-002: `typed_unavailable` Fail-Closed Claim Policy

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Treat mandatory missing evidence as `typed_unavailable` and fail closed by claim tier. Only clearly labeled exploratory documentation can survive; statistical significance, robustness, `paper_candidate` and `production_like` claims are blocked. |
| Alternatives | A. Warning-only `NEEDS_REVIEW`; B. binary missing/blocked for all tiers; C. manual waiver for paper candidates. |
| Pros | Prevents silent PASS; preserves truthful exploratory notes; creates deterministic release wording. |
| Cons | More historical artifacts will remain blocked until follow-up evidence producers exist. |
| Impact | Claim tier table, CR155 negative regression, CP7/CP8 wording and future fixture tests. |
| Risk | Too-strict blocking could frustrate exploratory workflows if wording is not clear. |
| Rollback / Switch | If CP3 reviewer wants a softer exploratory path, keep higher-tier blocks and adjust only exploratory wording. Do not allow `paper_candidate` or `production_like` bypass without a new waiver policy. |

## ADR-CR161-003: Integrate Through CR151/CR154, No Parallel Gate Family

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Extend CR151 statistical admission evidence refs and CR154 shared reliability gate artifact summaries. Do not create a separate CR161 admission gate family. |
| Alternatives | A. Directly extend only `StrategyAdmissionPackage`; B. create independent CR161 gate; C. replace CR151/CR154 gates. |
| Pros | Preserves existing closed foundations; keeps one admission vocabulary; reduces duplicate policy and release wording drift. |
| Cons | Requires future adapter discipline when implemented. |
| Impact | Module boundaries, dependency direction, Story planning if route changes, release wording and long-term maintenance. |
| Risk | CR151 and CR154 summaries may need small adapter updates later if implementation is authorized. |
| Rollback / Switch | If CR151/CR154 cannot represent a required object later, add adapter fields first. Only consider a new gate after a CP3 architecture change or new CR deprecates the old surfaces. |

## ADR-CR161-004: CR155 Negative Regression Only

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | verification |
| Recommendation | Use CR155 only to verify blocked status preservation: `blocked_admission_failed` remains true and `paper_candidate=false`. Missing historical trial lineage, p-values, PBO/DSR and fold metrics become `typed_unavailable`; they are not reconstructed. |
| Alternatives | A. Require CR155 to provide historical C1/C2 computable proof; B. omit CR155 from regression; C. turn CR155 into a remediation target. |
| Pros | Uses the strongest known negative sample without fabricating history; aligns with CP2. |
| Cons | Does not create positive evidence from CR155. |
| Impact | Verification wording, CP7 regression expectations, no-overclaim boundary. |
| Risk | Rerun consistency could be misread as admission readiness if blocked claims are not explicit. |
| Rollback / Switch | If CR155 evidence facts conflict with CP2, stop and reopen CP2 regression criteria. Do not repair CR155 inside CR161 without a new CR. |

## ADR-CR161-005: Five Follow-up Candidates for Computable Evidence

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | follow_up_tracking |
| Recommendation | Keep current CP3 slice design-only and define `FU-CR161-001` through `FU-CR161-005` for lineage instrumentation, multiple-testing/PBO/DSR computation, fold-level OOS, economic cost/impact and capacity/liquidity. |
| Alternatives | A. Implement all analytics in current CR; B. collapse all work into one follow-up; C. defer without named candidates. |
| Pros | Makes scope truthful; separates different data/model risks; gives host-orchestrator clean future routing options. |
| Cons | Requires follow-up governance before evidence can become `present`. |
| Impact | CP3 Decision Brief, future route plan, Feature design triggers and product backlog. |
| Risk | Follow-ups could be forgotten if not tracked by host-orchestrator. |
| Rollback / Switch | Promote any FU to an active CR only after user approval and explicit authorization for its implementation/data/runtime needs. |

## ADR-CR161-006: No New Runtime, Data, Credential, Implementation or Publish Authorization

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | security |
| Recommendation | Keep CR161 CP3 design-only. Do not access credentials, `.env`, real lake, NAS, providers, QMT/MiniQMT/xtquant/gateway runtime, broker/trading, external frameworks, Git remote write or publish. Do not modify source code or tests. |
| Alternatives | A. Authorize source/test implementation after CP3; B. authorize real lake validation; C. authorize external framework computation. |
| Pros | Matches user and CP2 authorization boundary; prevents methodology design from becoming runtime work. |
| Cons | No empirical proof in the current slice. |
| Impact | Work products limited to HLD, ADR, discussion log/checkpoint and CP3 result JSON. |
| Risk | Later users may assume CP3 approval authorizes implementation. |
| Rollback / Switch | Any implementation, real data or runtime action must go through CP3 approval plus CP4/CP5/CP6 or a separate CR/human authorization. |

## Design Confirmation Points

| Decision ID | Type | Recommended Decision | Alternatives | Impact / Risk | Rollback / Switch |
|---|---|---|---|---|---|
| `DQ-CP3-CR161-001` | architecture | Approve contract-first evidence availability overlay. | Compute-first implementation; documentation-only; parallel gate. | Controls whether current route stays CP3-only. | Switch after explicit route change. |
| `DQ-CP3-CR161-002` | architecture | Approve `typed_unavailable` fail-closed by claim tier. | Warning-only; binary block; waiver path. | Controls claim wording and safety. | Adjust exploratory wording only; higher tiers remain blocked. |
| `DQ-CP3-CR161-003` | architecture | Integrate through CR151/CR154 refs/summaries. | Parallel CR161 gate; package-only fields. | Controls module boundaries and maintenance. | Add adapters before creating any new gate. |
| `DQ-CP3-CR161-004` | verification | Preserve CR155 as blocked negative regression only. | Historical reconstruction; omit CR155; remediation target. | Controls regression truthfulness. | Reopen CP2 if CR155 facts conflict. |
| `DQ-CP3-CR161-005` | follow_up_tracking | Accept `FU-CR161-001`..`005` as deferred candidates. | Implement all now; collapse follow-ups. | Controls future route planning and scope. | Promote individual FU by explicit CR/approval. |
| `DQ-CP3-CR161-006` | security | Confirm no new implementation/runtime/data/credential/external/publish authorization. | Separate authorized implementation/data CR. | Prevents unsafe side effects. | Stop and request authorization if any forbidden action is needed. |

## Follow-up Candidate Register

| Follow-up ID | Title | Current Status |
|---|---|---|
| `FU-CR161-001` | Research-engine trial lineage instrumentation | candidate |
| `FU-CR161-002` | Multiple testing / PBO / DSR computable evidence implementation | candidate |
| `FU-CR161-003` | Walk-forward fold-level metrics and OOS evidence computation | candidate |
| `FU-CR161-004` | Economic cost / impact approximation implementation | candidate |
| `FU-CR161-005` | Capacity / liquidity sizing and alpha decay implementation | candidate |

## Consistency Notes

| Topic | Decision |
|---|---|
| Current slice semantics | `typed_unavailable` fail-closed contract semantics only; no computable FDR/PBO/DSR, no fold-level OOS computation, no real TCA, no capacity implementation. |
| CR151 integration | CR161 evidence availability feeds CR151 statistical evidence refs/summaries where statistical claims depend on those objects. |
| CR154 integration | CR161 evidence availability feeds CR154 shared reliability artifact refs/summaries and blocked claims. |
| CR155 | Keep `blocked_admission_failed` and `paper_candidate=false`; no historical reconstruction. |
| Blueprint artifacts | Standalone blueprint/domain/dependency maps are waived for current CP3-only slice and should be revisited before any implementation route. |

## Not Authorized

- source or test implementation
- research-engine forward instrumentation in the current slice
- simulation, paper, live or trading runtime
- QMT, MiniQMT, xtquant or gateway runtime
- broker operation or order write
- credential, secret or `.env` read
- new real lake read
- real lake write
- NAS read, write, sync or metadata normalization
- provider fetch
- catalog, store, registry, model or prediction write
- external framework clone, install or run
- Git remote write
- true release execution or publish

## Change Log

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-07-09 | Initial draft for CP3 review. |
