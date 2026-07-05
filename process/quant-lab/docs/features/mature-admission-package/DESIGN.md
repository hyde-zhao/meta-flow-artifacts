---
feature_id: "FEAT-17"
feature_name: "Mature Admission Package"
change_id: "CR-157"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-05T13:18:00+08:00"
owner: "host-orchestrator"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
implementation_allowed: false
authorization_boundary: "local/static/fixture-only planning; no LLD approval, source implementation, tests implementation, real lake/NAS/provider/QMT/runtime/simulation/paper/live/trading/broker/credential/feed/order/reconciliation/store/catalog/registry/publish"
---

# Mature Admission Package Feature Design

## Purpose

Define the CP4 implementation-design boundary for CR157 mature admission package builder work. The feature owns package assembly semantics for Stage 2 multifactor research exit and Stage 3 candidate admission handoff.

This document is a planning artifact only. It does not approve LLD, implementation, tests implementation, real data, runtime or publishing.

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-05 | host-orchestrator | Initial CR157 CP4 feature design for mature admission package builder. |

## Feature Boundary

| Area | In Scope | Out of Scope |
|---|---|---|
| Package builder | Assemble a deterministic admission package from existing Stage 2 refs, statuses, blocked claims and forbidden operation counters. | Reimplementing FactorSpec, FactorRunSpec, factor panel, label window, evaluation or portfolio/risk engines. |
| Completeness policy | Validate mandatory refs and fail closed on missing mandatory objects, stale refs, invalid hashes or unknown readiness. | Treating missing fields as PASS or silently defaulting to runtime readiness. |
| Status model | Package-level `PASS`, `FAIL`, `NEEDS_REVIEW`, `BLOCKED`; artifact-level `n/a-with-reason`. | Paper/live/trading readiness or broker readiness claims. |
| Compatibility | Consume CR150 / CR151 / CR154 outputs as refs. | Mutating closed CR150 / CR151 / CR154 baselines in CP4. |

## Mandatory Stage 2 Exit Refs

| Ref Family | Required Semantics |
|---|---|
| Factor contracts | `FactorSpec` and `FactorRunSpec` refs with version/hash/status. |
| Data slice | factor panel and label window refs with point-in-time and leakage guard status. |
| Evaluation | validation/evaluation report refs from existing statistical and reliability gates. |
| Portfolio/risk | portfolio/risk policy refs or `n/a-with-reason` when explicitly not applicable to the static slice. |
| Admission package | package summary, readiness status, blocked claims and forbidden operation counters. |
| Evidence index | refs-only evidence index ref linking package entries to source evidence. |

## Integration Contract

| Caller / Neighbor | Direction | Contract |
|---|---|---|
| FEAT-03 research loop | Consumed by package builder | Provides factor, label, evaluation and package source refs. |
| FEAT-18 evidence traceability | Required companion | Package entries must point to evidence index entries rather than embedding report bodies. |
| FEAT-19 handoff guardrails | Downstream consumer | Stage 2/3 handoff may only accept complete packages or blocked packages with explicit reasons. |
| FEAT-07 runtime safety | Guard provider | Any forbidden operation counter above zero forces package `BLOCKED`. |

## Failure Paths

| Failure | Required Behavior |
|---|---|
| Missing mandatory ref | Package `BLOCKED` with machine-readable blocked claim. |
| Unknown ref freshness or invalid hash | Package `BLOCKED` until ref is regenerated or marked `n/a-with-reason` by CP5-approved policy. |
| Nonzero forbidden operation counter | Package `BLOCKED`; cannot be downgraded in CP5. |
| Unsupported event/ML adapter field | Record deferred adapter ref and block only the adapter-specific claim, not unrelated Stage 2 static evidence. |

## Not Authorized

- No LLD approval before CP5.
- No source implementation or test implementation in CP4.
- No real lake read/write, NAS read/write/sync/restore, provider fetch, runtime, QMT, MiniQMT, xtquant, simulation, paper, live, trading, broker, credential, feed, order, reconciliation, store, catalog, registry, publish, external framework or Git remote action.
