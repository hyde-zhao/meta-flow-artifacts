---
discussion_id: "CP3-CR154-HLD-DISCUSSION"
cr_id: "CR-154"
phase: "solution-design"
agent_role: "meta-se"
status: "drafted-for-cp3"
created_at: "2026-07-03T00:28:00+08:00"
context_ref: "process/context/CP3-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-HLD-CONTEXT.yaml"
source_cp2: "process/checkpoints/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE.md"
---

# CP3 CR154 HLD Discussion Log

## Purpose

This log records the Architecture Gray Areas used to form the CR154 HLD before CP3 human review. It is an audit and recovery artifact only; downstream design should consume the HLD and ADR draft:

- `process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md`
- `process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md`

## Context Inputs

| Input | Path | Use |
|---|---|---|
| CP3 capsule | `process/context/CP3-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-HLD-CONTEXT.yaml` | Required CP3 constraints, AGQ list, no-runtime boundary. |
| CR summary | `process/changes/summaries/CR-154-CROSS-STRATEGY-RELIABILITY-GATES-2026-07-03.summary.json` | Lightweight CR state and CP3 design constraints. |
| CR body | `process/changes/CR-154-CROSS-STRATEGY-RELIABILITY-GATES-2026-07-03.md` | Approved CP2 scope and hard CP3 design constraints. |
| CP2 checkpoint | `process/checkpoints/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE.md` | Accepted CP2 decisions and no-runtime/no-real-data boundary. |
| UC baseline expansion | `process/USE-CASES.md#UC-58`, `#UC-59`, `#UC-60` | Required because CP3-DC-CR154-002 needs UC traceability and the capsule points to legacy `process/USE-CASES.md`. |
| REQ anchor expansion | `process/REQUIREMENTS.md` selected rows | Required because CP3-DC-CR154-002 requires visible REQ anchor policy. |
| Reference HLD/ADR formats | CR151/CR152/CR153 HLD/ADR under `process/docs/design/` | Format reference only; no content copied as CR154 design truth. |

## Read Expansion Log

| Path | full_doc_read_reason | Result |
|---|---|---|
| `process/USE-CASES.md` | Needed selected UC-58/59/60 anchors for CP3-DC-CR154-002 traceability. | Read selected sections and grep anchors only. |
| `process/REQUIREMENTS.md` | Needed existing REQ anchor candidates for CP3-DC-CR154-002. | Read targeted lines and selected anchors only. |
| `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` | Needed CR154 roadmap relation and CR153 slot delegation evidence. | Read grep hits only. |
| `docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md` | Needed cross-strategy production reliability scope and CR153 delegation evidence. | Read grep hits only. |
| `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` | Needed statistical reliability and impact model gap source. | Read grep hits only. |

## Architecture Gray Areas Summary

| AGQ ID | Topic | Decision used for HLD | Basis |
|---|---|---|---|
| AGQ-CR154-001 | Cross-strategy gate shape | Shared contract plus strategy-specific adapters. | CP2 accepted cross-strategy scope; avoids three incompatible gate families. |
| AGQ-CR154-002 | Statistical reliability artifact model | Cross-strategy artifact refs with CR151/CR152/CR153 contributors; do not collapse to plain trap status. | CP3-DC-CR154-001 and E2E review C1/XC-GAP-2/3. |
| AGQ-CR154-003 | Admission default policy | Tier table with opt-in, default-required and release-blocking modes. | CP3-DC-CR154-003 and CP2 DQ accepted default policy. |
| AGQ-CR154-004 | No-runtime reconciliation boundary | Slots/status/refs only; real reconciliation deferred to runtime authorization. | CP2 no-runtime/no-real-data decision. |
| AGQ-CR154-005 | REQ anchoring policy | Reuse existing REQ anchors for CP3, add CR154-specific REQ entries later only if CP3 review requests product-baseline refresh. | CP3-DC-CR154-002; avoids editing REQUIREMENTS in CP3. |
| AGQ-CR154-006 | CR153 universe slot lifecycle | Keep CR153 `universe_pit_audit` as adapter/source ref in first wave; CR154 owns shared PIT universe gate semantics. | CP3-DC-CR154-004. |
| AGQ-CR154-007 | Impact enum and strategy-specific n/a | Controlled enum: `square_root`, `almgren_chriss`, `gatheral`, `custom`, `n/a-with-reason`; ML-only fields are strategy-specific n/a for non-ML. | CP3-DC-CR154-005 and CP3-DC-CR154-006. |

## Advisor Tables

### AGQ-CR154-001: Cross-Strategy Gate Shape

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Shared reliability gate contract plus adapters | One policy language for multifactor, ML and event-driven; avoids duplicated release wording; lets each strategy add evidence refs. | Requires adapter boundary discipline. | Module boundaries, admission package, validation, release blocking. | Recommended. | Keep while CR151/152/153 share the same admission vocabulary; switch if one strategy requires incompatible status semantics. |
| B. Separate gate objects per strategy | Local names are simple; no shared abstraction up front. | Three implementations drift; default policy becomes ambiguous. | Story planning, maintenance, CP7 verification, release wording. | Not recommended. | Use only if CP5 proves common gate semantics cannot represent a strategy family. |
| C. Documentation-only gate checklist | Minimal implementation cost. | Not machine-verifiable; cannot block overclaims. | Release claims, CP7 evidence, user trust. | Rejected. | Only acceptable as a temporary CP3 note if implementation is cancelled. |

### AGQ-CR154-002: Statistical Reliability Artifact Model

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Explicit artifact refs model | Covers multiple-testing, FDR/BH, WRC/SPA, PBO/CSCV, DSR/deflation, trials, OOS, purge/embargo and blocked claims. | Larger contract than a single status enum. | Data structures, admission evaluator, fixtures, CP3-DC-CR154-001. | Recommended. | Switch to narrower model only if CP3 accepts loss of auditability, which is not recommended. |
| B. Reuse CR151 statistical gate summary only | Low duplication with CR151. | ML/event/PIT/capacity refs are squeezed into CR151-specific shape; CR154 artifacts remain implicit. | CR151 compatibility, event and ML adapters. | Not recommended as sole model; useful as contributor. | Use CR151 report refs as inputs to the shared artifact model. |
| C. Plain trap status enum | Easy to implement. | Violates CP3-DC-CR154-001; hides unavailable evidence. | CP3 review, release blocking, future verification. | Rejected. | No valid first-wave switch condition. |

### AGQ-CR154-003: Admission Default Policy

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Tier table by strategy class, release profile, risk level and evidence completeness | CP5 can split Stories directly; release wording deterministic. | Requires careful compatibility exceptions. | Existing callers, release text, gate evaluator, CP3-DC-CR154-003. | Recommended. | Switch tier defaults if CP5 compatibility tests show existing package-visible callers break. |
| B. Immediately make all gates release-blocking | Strongest reliability posture. | High compatibility risk; fixture-only gaps could block too broadly. | CR151/152/153 callers, release readiness. | Not recommended first wave. | Use for production/paper/live release profiles after evidence completeness is high. |
| C. Keep everything opt-in | No caller disruption. | New gates can be bypassed; user goal not met. | Release claims and human gate trust. | Rejected for CR154 default path. | Keep only for exploratory/not-for-release profiles. |

### AGQ-CR154-004: No-Runtime Reconciliation Boundary

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Structured slots with status, refs and n/a-with-reason | Future paper/live readiness can consume stable slots without runtime side effects. | Does not prove real reconciliation. | Security boundary, future runtime CR, release wording. | Recommended. | Switch only under a separate runtime authorization gate. |
| B. Implement real reconciliation now | Closer to full production. | Violates CR154 CP2 boundary. | Broker/runtime/security. | Rejected. | Requires new runtime_authorization decision or separate CR. |
| C. Omit reconciliation until runtime work | Keeps scope small. | Loses planned integration points; later retrofit risk. | Future paper/live readiness. | Not recommended. | Use if CP3 narrows CR154 to statistical gates only. |

### AGQ-CR154-005: Requirement Traceability and CR154 REQ Anchoring

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Reuse existing REQ anchors with explicit HLD/ADR traceability | No REQUIREMENTS edit in CP3; keeps product baseline stable; still auditable. | CR154-specific REQ IDs remain absent. | CP3 review, later Story acceptance. | Recommended. | Add CR154-specific REQ IDs if CP3 review demands product-baseline refresh or anchor gaps are found. |
| B. Add CR154-specific REQ entries before HLD | Highest trace granularity. | Violates current user task boundary and reopens product docs during CP3. | Product baseline, CP2 scope, workflow state. | Not recommended in this delegated turn. | Use via host-orchestrator if CP3 requests changes. |
| C. Implicitly rely on UC/roadmap only | Quickest. | Violates CP3-DC-CR154-002. | CP3 consistency. | Rejected. | No valid first-wave switch condition. |

### AGQ-CR154-006: CR153 `universe_pit_audit` Upgrade Lifecycle

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Retain CR153 slot as adapter/source ref; CR154 owns shared PIT universe gate | Backward compatible; prevents duplicate owner; supports first-wave migration. | Requires explicit owner boundary in Story planning. | CR153 compatibility, file ownership, CP3-DC-CR154-004. | Recommended. | Switch to deprecation only after CR154 gate has stable adapter coverage and release migration evidence. |
| B. Delete CR153 slot and replace with CR154 gate | Simplifies final shape. | Forbidden in first wave; breaks CR153 semantics. | Compatibility, release notes. | Rejected. | Only after a later migration CR. |
| C. Keep both as independent owners | No migration work. | Duplicate gate semantics and conflicting results. | Admission package, verification. | Rejected. | No valid first-wave switch condition. |

### AGQ-CR154-007: Impact Model Enum and Strategy-Specific N/A Policy

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Controlled enum plus n/a-with-reason policy | Machine-checkable; avoids real TCA overclaim; supports custom with rationale. | Cannot calibrate true impact without real execution data. | Capacity contract, release wording, CP3-DC-CR154-005/006. | Recommended. | Switch to calibrated TCA only under real execution data authorization. |
| B. Free text model family | Flexible. | Unverifiable and inconsistent. | CP7 tests, release wording. | Rejected. | No valid first-wave switch condition. |
| C. Require all strategy families to implement ML-only methods | Uniform table shape. | Misclassifies triple-barrier/meta-labeling and feature importance for non-ML. | Multifactor/event adapters, release blockers. | Rejected. | No valid switch; use strategy-specific n/a instead. |

## Scenario Simulation Inputs

| Scenario | Recommendation result |
|---|---|
| UC-58 multifactor release gate | Multifactor adapter contributes CR151 statistical refs plus PIT universe, capacity/impact and admission policy evidence; missing PBO/DSR or PIT universe refs block production wording. |
| UC-59 ML admission | ML adapter contributes purged/embargo split refs, trial counts, model/training snapshot refs and ML-specific fields; triple-barrier/meta-labeling can be active only for ML and remains CR152/follow-up if not implemented. |
| UC-60 event-driven admission | Event adapter keeps event time semantics from CR153 and delegates `universe_pit_audit` to CR154 PIT gate owner while preserving the CR153 source slot. |

## Pending CP3 Decisions

| Decision ID | Type | Recommendation | Alternatives | Risk / Impact | Rollback / Switch |
|---|---|---|---|---|---|
| CP3-DQ-CR154-SHARED-CONTRACT | architecture | Approve shared reliability gate contract plus adapters. | Per-strategy gates; documentation-only. | Affects module boundaries and Story ownership. | Split per strategy if CP5 proves incompatible semantics. |
| CP3-DQ-CR154-REQ-ANCHOR-POLICY | architecture | Reuse existing REQ anchors in CP3; add CR154-specific REQ later only if review requires. | Add REQ now; implicit anchors only. | Affects traceability and product baseline churn. | Host creates product-baseline refresh if CP3 changes requested. |
| CP3-DQ-CR154-DEFAULT-POLICY | architecture | Approve HLD §8 admission tier table as the default gate policy. | Keep all gates opt-in; make all gates release-blocking. | Affects release wording, compatibility and CP5 Story split boundaries. | Adjust T1/T2 defaults during CP5 if compatibility requires. |
| CP3-DQ-CR154-NO-RUNTIME | security | Keep all gates local/static/fixture-only; no real lake/NAS/provider/QMT/runtime/broker/credential/feed/reconciliation/publish. | Separate runtime_authorization CR. | Prevents overclaim; limits proof to contract semantics. | New runtime/data authorization gate. |
