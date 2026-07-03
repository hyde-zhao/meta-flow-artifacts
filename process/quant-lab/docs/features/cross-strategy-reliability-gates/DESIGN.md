---
feature_id: "FEAT-15"
feature_name: "Cross-Strategy Production Reliability Gates"
change_id: "CR-154"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-03T10:40:00+08:00"
owner: "host-orchestrator"
source_hld: "process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
implementation_allowed: false
authorization_boundary: "local/static/fixture-only; no LLD approval, source implementation, tests implementation, real lake/NAS/provider/QMT/runtime/simulation/paper/live/trading/broker/credential/feed/order/reconciliation/store/catalog/registry/publish"
---

# Cross-Strategy Production Reliability Gates Feature Design

## Purpose

Define the CP4 implementation-design boundary for CR154 before Story LLD work starts. This feature owns the first-wave shared reliability gate contract, strategy adapter mapping, gate-specific evidence policies, admission default policy and compatibility hooks for CR151 / CR152 / CR153.

This document is a planning artifact only. It does not approve LLD, source changes, test implementation, real runtime, real data, broker access or publishing.

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-03 | host-orchestrator | Initial CP4 feature design. Incorporates Story split review findings: Gate 5 explicit Story, Phase A runnable fixture schema ownership and Gate 6 tier resolution full-lld. |

## Feature Boundary

| Area | In Scope | Out of Scope |
|---|---|---|
| Shared contract | Gate summary, artifact refs, four-state status, blocked claims, release-blocking reason, first runnable fixture schema. | Runtime execution, live data reads, provider integration. |
| Strategy adapters | Multifactor, ML and event-driven mapping into the shared contract. | Replacing CR151 / CR152 / CR153 source semantics before LLD approval. |
| Gate evidence policies | Gate 1 statistical artifacts, Gate 2 CV governance, Gate 3 PIT universe, Gate 4 capacity/impact/liquidity, Gate 5 regime/attribution/reconciliation slots. | Real statistical calibration, real TCA, real universe construction, real reconciliation. |
| Admission default policy | Tier resolver and release wording semantics for opt-in / default-required / release-blocking / not-authorized. | Paper/live/trading readiness, broker readiness, true release execution. |
| Compatibility | CR153 `universe_pit_audit` lifecycle, CR152 ML-only n/a policy, CR151/152/153 admission summary consumption. | Deleting existing CR153 slot or silently changing historical CR semantics. |

## Story Split Decisions

| Review Finding | CP4 Treatment |
|---|---|
| Gate 5 was not explicit in HLD Phase B. | Add `CR154-S06-regime-attribution-reconciliation-slots` as a first-wave full-lld Story. |
| Phase A/B fixture schema ownership was ambiguous. | Put shared fixture schema and first runnable fixture in `CR154-S01-shared-gate-contract-fixture-skeleton`; Gate-specific fixtures extend it later. |
| Gate 6 tier resolution was under-specified. | Add `CR154-S07-admission-default-policy-tier-resolution` as a full-lld Story with resolver function/config boundary, fail-closed unknown profile behavior and release wording AC. |
| B1 statistical artifact workload is large. | Keep as one Story but require CP5 LLD to split adapter subtasks by strategy family and include 12 artifact fields, severity mapping and propagation cases. |

## Shared Contract Shape For CP5

CP5 LLD must define concrete names and serialization, but the following conceptual fields are mandatory:

| Field Family | Required Semantics |
|---|---|
| Gate identity | `gate_id`, `gate_name`, `strategy_class`, `release_profile`, `risk_level`, `evidence_completeness`. |
| Status | `PASS`, `FAIL`, `NEEDS_REVIEW`, `BLOCKED`, plus `n/a-with-reason` at artifact level. |
| Evidence refs | Structured refs with `artifact_type`, `ref`, `source_cr`, `owner_gate`, `status`, `n/a_reason`. |
| Blocked claims | Claim id, reason, source gate, release wording impact and unlock condition. |
| Release blocking | `release_blocking_reason`, gate mode and user-visible wording. |
| Forbidden operations | Counters for real lake/NAS/provider/runtime/broker/credential/feed/order/store/catalog/registry/publish operations; any nonzero count is `BLOCKED`. |

## Gate-Specific Design Requirements

| Gate | Story | Required CP5 Design Content |
|---|---|---|
| Gate 1 Backtest trap / statistical reliability | CR154-S02 | Multiple-testing correction refs, FDR/BH, WRC/SPA, PBO/CSCV, DSR/deflation, trial counts, OOS refs, purge/embargo refs, survivorship refs, impact/capacity refs, blocked claims and severity mapping by release profile. |
| Gate 2 CV governance | CR154-S03 | Shared walk-forward / OOS / purged-embargo contract and strategy-specific adapter mapping without copying CR152 ML-only semantics into multifactor/event strategies. |
| Gate 3 PIT universe | CR154-S04 | Survivorship-free / PIT universe gate, CR153 `universe_pit_audit` delegated source lifecycle, no real universe construction. |
| Gate 4 Capacity / impact / liquidity | CR154-S05 | ADV participation, capacity dollars, cost-underestimation status, liquidity sizing refs, `impact_model_family` enum: `square_root`, `almgren_chriss`, `gatheral`, `custom`, `n/a-with-reason`. |
| Gate 5 Regime / attribution / reconciliation | CR154-S06 | Structured slots, refs, status and `n/a-with-reason`; no runtime/broker reconciliation. |
| Gate 6 Admission default policy | CR154-S07 | Tier resolver from HLD §8, unknown profile fail-closed, release wording and compatibility behavior. |

## Follow-Through Hooks

| Hook ID | Feature Treatment |
|---|---|
| FT-CR154-CP5-001-WRC-SPA-SEVERITY | Owned by S02; CP5 must define missing WRC/SPA severity by release profile and claim type. |
| FT-CR154-CP5-002-CROSS-GATE-PROPAGATION | Owned by S02/S04/S05; CP5 fixtures must prove Gate 3/4 blocked states propagate into Gate 1 refs, blocked claims and release-blocking reason. |
| FT-CR154-CP5-003-NUMERIC-THRESHOLDS | Owned by S02/S05/S07; CP5 must define defaults, config ownership or explicit `n/a-with-reason` policy for DSR/PBO/capacity/risk-level thresholds. |
| FT-CR154-CP5-004-MF-GAP-2-6-7-DEFERRED | Owned by S08; CP5 mapping must preserve that MF-GAP-2/6/7 are deferred to factor-evaluation follow-up CR. |
| FT-CR154-CP5-005-REQ-ANCHOR-PRESERVATION | Owned by all Stories; CP5 must preserve HLD §11 UC/REQ mapping or route product-baseline refresh before implementation. |

## Integration Contract

| Caller / Neighbor | Direction | Contract |
|---|---|---|
| CR151 statistical gate | Consumed by CR154 adapters | CR154 must consume statistical refs and status; it must not weaken CR151 fail-closed behavior. |
| CR152 ML gate | Consumed by CR154 ML adapter | CR154 may mark ML-only fields for non-ML strategies as `n/a-with-reason`; unfinished ML method work remains CR152 follow-up. |
| CR153 event gate | Consumed by CR154 event adapter | CR153 `universe_pit_audit` remains a source slot; CR154 owns shared PIT release-blocking wording. |
| `StrategyAdmissionPackage` | Consumes CR154 summary | Package gets summary, blocked claims and release wording only; it does not gain runtime or trading authority. |

## Failure Paths

| Failure | Required Behavior |
|---|---|
| Missing mandatory artifact | `BLOCKED` for release-blocking profiles or `NEEDS_REVIEW` only where tier table explicitly allows it. |
| Unknown release profile | Fail closed until classified. |
| Nonzero forbidden operation counter | `BLOCKED`; cannot be waived in CP5/CP6/CP7. |
| Strategy-specific method not applicable | Must be explicit `n/a-with-reason`; omission is not acceptable. |
| Gate 3/4 blocked but Gate 1 refs clean | CP5 must treat this as cross-gate propagation failure. |

## Not Authorized

- No LLD approval before CP5.
- No source implementation or test implementation in CP4.
- No real lake read/write, NAS read/write/sync/restore, provider fetch, runtime, QMT, MiniQMT, xtquant, simulation, paper, live, trading, broker, credential, feed, order, reconciliation, store, catalog, registry, publish, external framework or Git remote action.
