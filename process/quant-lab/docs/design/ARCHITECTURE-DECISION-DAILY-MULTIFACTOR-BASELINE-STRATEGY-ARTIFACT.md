---
status: "approved-cp3"
version: "0.1"
cr_id: "CR-155"
title: "ADR: Daily Multifactor Baseline Strategy Artifact"
source_hld: "docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-04T18:37:08+08:00"
authorization_boundary: "CR155-scoped local governed lake/current truth readonly only; no write/runtime/trading/publish"
---

# ADR: CR155 Daily Multifactor Baseline Strategy Artifact

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-04 | host-orchestrator | Initial CP3 ADR draft for artifact ownership, readonly pipeline, admission composition, rerun consistency and no-runtime boundary. |
| 0.2 | 2026-07-04 | host-orchestrator | CP3 approved by user; ADR decisions are confirmed for CP4 Story planning and CP5 design evidence preparation. |

## ADR-CR155-001: Standalone Strategy Artifact Contract

| Field | Decision |
|---|---|
| Status | accepted-cp3 |
| Type | architecture |
| Recommendation | Create a standalone daily multifactor baseline strategy artifact and let the admission package consume its summary and evidence refs. |
| Alternatives | Report-pack-only; extend only `StrategyAdmissionPackage`. |
| Pros | Stable audit surface, clear ownership, reusable downstream refs. |
| Cons | Requires explicit contract and CP5 story split. |
| Rollback / Switch | If scope is too large, keep required identity/policy/gate refs and defer optional report details. |

## ADR-CR155-002: CR155-Scoped Readonly Validation Pipeline

| Field | Decision |
|---|---|
| Status | accepted-cp3 |
| Type | architecture |
| Recommendation | Use an isolated readonly adapter for CP2-approved local governed lake/current truth validation. |
| Alternatives | Fixture-only; broader direct data lake integration. |
| Pros | Enables real local baseline evidence while preserving no-write/no-runtime boundary. |
| Cons | Needs strict provenance and forbidden-operation checks. |
| Rollback / Switch | If readonly provenance cannot be proven, CP5 must switch to fixture/static and forbid real-data claims. |

## ADR-CR155-003: Separate Statistical, Reliability and Package Decisions

| Field | Decision |
|---|---|
| Status | accepted-cp3 |
| Type | architecture |
| Recommendation | Preserve CR151 statistical admission status, CR154 reliability status and final admission package decision as separate refs. |
| Alternatives | Single PASS/FAIL; free-text package note. |
| Pros | Keeps failure reasons visible and prevents gate result collapse. |
| Cons | More fields and report refs. |
| Rollback / Switch | Package may expose a summary field, but must retain component refs. |

## ADR-CR155-004: Mandatory Rerun Consistency Evidence

| Field | Decision |
|---|---|
| Status | accepted-cp3 |
| Type | implementation |
| Recommendation | Require two reruns and compare return, drawdown, turnover, cost, capacity/liquidity summary and gate status. |
| Alternatives | Single run; ad hoc manual explanation. |
| Pros | Detects nondeterminism and supports auditability. |
| Cons | Adds runtime cost in CP6/CP7 validation. |
| Rollback / Switch | CP5 may define metric-specific tolerances, but cannot remove rerun evidence without CP3/CP5 change. |

## ADR-CR155-005: No Runtime / No Write Boundary

| Field | Decision |
|---|---|
| Status | accepted-cp3 |
| Type | security |
| Recommendation | CP3 does not authorize Story decomposition before approval, LLD, source implementation, tests, lake write, NAS/provider/credential/runtime/trading/broker/catalog/store/registry/publish. |
| Alternatives | Separate runtime/data authorization CR. |
| Pros | Preserves audit and safety boundary. |
| Cons | CP3 cannot prove production or trading readiness. |
| Rollback / Switch | Any runtime/data/write need must open a separate human gate. |

## CP3 Decision Items

| Decision ID | Type | Recommended Decision |
|---|---|---|
| CP3-DQ-CR155-ARTIFACT-CONTRACT | architecture | Approve standalone artifact contract plus admission package consumer relationship. |
| CP3-DQ-CR155-READONLY-PIPELINE | architecture | Approve isolated CR155-scoped readonly validation pipeline. |
| CP3-DQ-CR155-RERUN-POLICY | implementation | Require two-rerun consistency evidence. |
| CP3-DQ-CR155-NO-RUNTIME | security | Confirm no runtime/write/trading/publish boundary. |
