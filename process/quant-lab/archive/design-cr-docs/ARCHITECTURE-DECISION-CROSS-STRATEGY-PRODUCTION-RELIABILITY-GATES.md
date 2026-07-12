---
status: "approved"
version: "0.1"
cr_id: "CR-154"
title: "ADR: Cross-Strategy Production Reliability Gates"
source_hld: "process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-03T01:05:00+08:00"
authorization_boundary: "design-only/static-only; no real lake/NAS/provider/QMT/runtime/simulation/paper/live/trading/broker/credential/external framework/catalog/event/model registry/feed/order/reconciliation/publish"
---

# ADR: CR154 Cross-Strategy Production Reliability Gates

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-03 | meta-se | Initial CP3 ADR draft covering shared contract/adapters, statistical artifacts, PIT universe ownership, impact enum, admission policy, no-runtime boundary and REQ anchor policy. |

## Agent / Skill Composition

| Role / Skill | Use in CR154 CP3 |
|---|---|
| `meta-se` | Owns Architecture Gray Areas, HLD and ADR draft. |
| `hld-designer` | Used for HLD structure, candidate comparison, traceability, scenario simulation and CP3 design checks. |
| `checkpoint-manager` | Used for CP3 consistency precheck shape and decision item structure. |
| `blueprint-design` | Not separately invoked in this delegated turn; HLD §2 records blueprint applicability and uses current roadmap/remediation/CR151-153 HLDs as active boundary baseline. |

## Platform Adaptation Differences

CR154 CP3 is a design-only change. It does not modify Python dependencies, platform installation, Agent files, Skill files, runtime commands, data lake paths, NAS routes, QMT gateway behavior, broker integration, catalog pointers, event/model registries or release publishing paths.

## ADR-CR154-001: Shared Contract plus Strategy-Specific Adapters

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Create one shared cross-strategy reliability gate contract, with multifactor, ML and event-driven adapters contributing evidence. |
| Alternatives | A. per-strategy gates; B. extend only `StrategyAdmissionPackage`; C. documentation-only checklist. |
| Pros | Consistent release-blocking semantics; lower long-term maintenance; avoids three incompatible gate policy vocabularies; supports CP3-DC-CR154-003 tier policy. |
| Cons | Requires explicit adapter boundaries and compatibility planning. |
| Impact | Module ownership, CP4/CP5 planning, release wording, compatibility with CR151/152/153. |
| Risks | Adapter might become too broad if strategy-specific fields leak into shared gate. |
| Rollback / Switch | If CP5 proves common status semantics cannot represent a strategy family, keep shared blocked-claim/release wording but split that strategy's evidence adapter into an independent gate. |

## ADR-CR154-002: Explicit Statistical Reliability Artifact Model

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Define a cross-strategy statistical artifact model with refs or `n/a-with-reason` for multiple-testing correction, FDR/BH, WRC/SPA, PBO/CSCV, DSR/Sharpe or IC deflation, trial_count/effective_trials, OOS split, purge/embargo, survivorship audit, impact/capacity, blocked claims and release-blocking reason. |
| Alternatives | A. reuse only CR151 statistical gate summary; B. plain trap status enum; C. defer statistical artifacts to runtime. |
| Pros | Satisfies CP3-DC-CR154-001; prevents hidden overclaim; creates machine-checkable missing-evidence paths. |
| Cons | Wider contract and more fixture cases later. |
| Impact | Gate evaluator, adapter mapping, CP5 design evidence, CP7 fixture validation, release wording. |
| Risks | Too many optional refs could weaken enforcement if n/a reasons are not validated. |
| Rollback / Switch | If scope must shrink, keep artifact names and mark lower-priority methods n/a-with-reason rather than deleting fields. |

## ADR-CR154-003: CR153 `universe_pit_audit` Compatibility and CR154 PIT Universe Ownership

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Retain CR153 `universe_pit_audit` as a first-wave adapter/source ref while CR154 owns shared PIT universe/survivorship gate semantics and release-blocking wording. |
| Alternatives | A. delete CR153 slot immediately; B. keep CR153 and CR154 as independent owners; C. defer PIT universe gate. |
| Pros | Backward compatible; satisfies CP3-DC-CR154-004; avoids duplicate policy owner. |
| Cons | Requires explicit delegation status in adapter mapping. |
| Impact | CR153 compatibility, Story ownership, admission package display, conflict handling. |
| Risks | If both CR153 and CR154 report separate PIT statuses, users may see contradictory release reasons. |
| Rollback / Switch | Later deprecation is allowed only after migration evidence proves all CR153 consumers use the CR154 gate; first wave must not delete CR153 compatibility semantics. |

## ADR-CR154-004: Controlled `impact_model_family` Enum and No Real TCA

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | implementation |
| Recommendation | Use controlled enum `square_root`, `almgren_chriss`, `gatheral`, `custom`, `n/a-with-reason`; require `no_real_tca_claim` in first wave. |
| Alternatives | A. free text impact model family; B. real TCA calibration now; C. commission/tax/slippage only. |
| Pros | Satisfies CP3-DC-CR154-005; machine-checkable; supports future calibrated models without overclaim. |
| Cons | Enum does not prove impact parameters are empirically calibrated. |
| Impact | Capacity/impact contract, release wording, fixture validation. |
| Risks | `custom` could become an unreviewed escape hatch. |
| Rollback / Switch | `custom` must include rationale, inputs, validation boundary and wording limits; true TCA requires separate real execution data authorization. |

## ADR-CR154-005: Admission Default Policy Tier Table

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Adopt HLD §8 tier table as default policy: exploratory remains opt-in with blocked claims; admission candidates are default-required; production-like/paper/live/trading profiles are release-blocking or not authorized. |
| Alternatives | A. immediately all release-blocking; B. keep all opt-in; C. caller decides ad hoc. |
| Pros | Satisfies CP3-DC-CR154-003; balances compatibility and reliability; gives CP5 direct split boundaries. |
| Cons | Requires compatibility tests around existing package-visible callers later. |
| Impact | Existing admission behavior, release wording, human gate decisions and future Story planning. |
| Risks | Too aggressive defaults could break historical exploratory workflows; too weak defaults could allow bypass. |
| Rollback / Switch | Roll T1/T2 thresholds back to opt-in/default-required split if CP5 compatibility evidence shows breakage; unknown profiles remain fail-closed. |

## ADR-CR154-006: No-Runtime / No-Real-Data Boundary

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | security |
| Recommendation | CR154 CP3 and first-wave design remain local/static/fixture-only; no real lake/NAS/provider/QMT/runtime/simulation/paper/live/trading/broker/credential/external framework/catalog/event-store/model-registry/feed/order/reconciliation/publish action is authorized. |
| Alternatives | A. authorize real lake readonly validation; B. authorize event feed/listener smoke; C. authorize broker or reconciliation probe. |
| Pros | Preserves CP2 security boundary; prevents fixture evidence from becoming runtime claim. |
| Cons | Cannot prove empirical production reliability or real TCA. |
| Impact | CP5/CP6/CP7 evidence mode, release wording, user authorization prompts. |
| Risks | Users may ask why reliability gates do not run real data; wording must remain clear. |
| Rollback / Switch | Any real operation requires a separate runtime_authorization or data authorization CR with explicit human approval. |

## ADR-CR154-007: REQ Anchor Policy for CP3

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Reuse existing REQ anchors for CP3 traceability and do not edit `process/REQUIREMENTS.md` in this design turn; add CR154-specific REQ IDs only if CP3 review requests product-baseline refresh or traceability gaps are found. |
| Alternatives | A. add CR154-specific REQ entries now; B. rely only on UC/roadmap without REQ mapping. |
| Pros | Satisfies CP3-DC-CR154-002 without reopening product baseline during CP3; preserves auditability. |
| Cons | CR154-specific REQ IDs are absent until a follow-up refresh. |
| Impact | CP3 traceability, CP4/CP5 acceptance mapping, product documentation churn. |
| Risks | A reviewer may prefer dedicated CR154 REQ rows before implementation. |
| Rollback / Switch | Host-orchestrator can route a product-baseline refresh before CP5 if CP3 is changes_requested on this point. |

## Strategy-Specific N/A Decision

| Field | Decision |
|---|---|
| Status | proposed-for-cp3 |
| Type | architecture |
| Recommendation | Triple-barrier/meta-labeling and feature importance are ML-only method fields. Multifactor and event-driven adapters must mark them `n/a-with-reason: ML-specific method, not applicable`; unfinished ML method work remains CR152/follow-up and is not a CR154 cross-strategy release blocker for non-ML strategies. |
| Alternatives | A. require all strategies to populate these fields; B. omit them from non-ML adapters. |
| Pros | Satisfies CP3-DC-CR154-006; prevents false blockers; preserves machine-visible n/a reasoning. |
| Cons | Requires explicit adapter validation for n/a reasons. |
| Impact | Multifactor/event release checks, ML follow-up tracking, CP5 acceptance criteria. |
| Rollback / Switch | If a non-ML strategy later uses ML labels/features, it must declare a hybrid strategy class and activate the ML adapter branch. |

## Design Confirmation Points

| Decision ID | Type | Recommended Decision | Alternatives | Impact / Risk | Rollback / Switch |
|---|---|---|---|---|---|
| CP3-DQ-CR154-SHARED-CONTRACT | architecture | Approve shared contract + adapters. | Per-strategy gates; documentation-only. | Affects module boundaries and long-term consistency. | Split adapter if common semantics fail. |
| CP3-DQ-CR154-REQ-ANCHOR-POLICY | architecture | Reuse existing REQ anchors for CP3; add CR154-specific later only if requested. | Add REQ now; implicit-only trace. | Balances traceability and process churn. | Product-baseline refresh before CP5. |
| CP3-DQ-CR154-DEFAULT-POLICY | architecture | Approve HLD §8 tier table. | All opt-in; all release-blocking. | Affects caller compatibility and release wording. | Adjust T1/T2 defaults during CP5 if compatibility requires. |
| CP3-DQ-CR154-NO-RUNTIME | security | Confirm no runtime/no real data/no broker/no publish boundary. | Separate runtime/data authorization CR. | Prevents unsafe overclaim; limits validation to fixtures. | New authorization gate. |

## Change Log

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-07-03 | Initial draft for CP3 review. |
