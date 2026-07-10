---
discussion_id: "CP3-CR161-HLD-DISCUSSION"
cr_id: "CR-161"
phase: "solution-design"
status: "ready-for-cp3-hld-review"
created_at: "2026-07-09T22:24:20+08:00"
context_ref: "process/context/CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-DESIGN-CONTEXT.yaml"
hld_ref: "docs/design/HLD-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING.md"
adr_ref: "docs/design/ARCHITECTURE-DECISION-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING.md"
discussion_checkpoint_ref: "process/checks/CP3-CR161-DISCUSSION-CHECKPOINT.json"
selection_basis: "CP2 approved decisions plus current user hard requirements for CR161 CP3 design"
---

# CP3 CR161 HLD Discussion Log

## Scope of This Discussion

This log records the architecture gray-area inputs used to form the CR161 CP3 HLD. The current turn did not spawn separate reviewer lanes and does not claim independent lane execution. The selected options come from:

- CP2 approved decisions in `process/checkpoints/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.md`.
- CP2 result facts in `process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json`.
- CP3 context in `process/context/CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-DESIGN-CONTEXT.yaml`.
- Current user hard requirements for the 7-object matrix, follow-up candidates, CR151/CR154 integration and CR155 negative regression.

## Architecture Gray Areas

| Gray Area ID | Question | Why It Matters | Selected Option | Canonical Refs | Status |
|---|---|---|---|---|---|
| `AGA-CR161-001` | Should current Wave 1 be contract-first or compute-first? | Determines whether CP3 stays design-only or routes into research-engine instrumentation and analytics computation. | Contract-first evidence availability overlay; missing lineage/evidence becomes `typed_unavailable`. | `DQ-CP2-CR161-WAVE1-TRIAL-LINEAGE-SOURCE`, CP3 context `approved_cp2_decisions` | selected |
| `AGA-CR161-002` | What is the claim ceiling for `typed_unavailable`? | Determines whether missing mandatory evidence blocks admission or merely warns. | Fail closed for mandatory evidence; exploratory limitation only when explicitly labeled. | `DQ-CP2-CR161-TYPED-UNAVAILABLE-FAIL-CLOSED` | selected |
| `AGA-CR161-003` | Should CR161 extend CR151/CR154 or create a new gate family? | Determines module boundary, dependency direction and long-term maintenance cost. | Extend CR151/CR154 evidence refs/summaries; no parallel gate family. | CP3 context `integration_boundary`, CR151/CR154 summaries | selected |
| `AGA-CR161-004` | How should CR155 be used? | Determines whether historical evidence gaps are handled truthfully. | Preserve CR155 as blocked negative regression only; do not reconstruct missing C1/C2 facts. | `DQ-CP2-CR161-CR155-NEGATIVE-REGRESSION` | selected |
| `AGA-CR161-005` | Which analytics are in current scope versus follow-up? | Determines whether HLD overpromises computation. | Current slice is typed-unavailable contract semantics; analytics are `FU-CR161-001`..`005`. | `DQ-CP2-CR161-FOLLOWUP-SPLIT`, CP3 context `follow_up_candidates` | selected |
| `AGA-CR161-006` | What authorization boundary applies? | Determines whether design may touch source, tests, credentials, data or runtime. | No new implementation, runtime, data, credential, external framework, Git remote or publish authorization. | `DQ-CP2-CR161-AUTHORIZATION-BOUNDARY`, CP3 context `authorization_boundary` | selected |

## Advisor Tables

### AGA-CR161-001: Contract-First vs Compute-First

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Contract-first evidence availability overlay | Fits approval; no code/runtime/data; makes missing evidence fail closed; safe for CR155. | Does not compute evidence values. | Scope, authorization, HLD/ADR, CP7 wording | Recommended | Switch only if user authorizes CP4/CP5/CP6 implementation route. |
| Compute-first implementation | Can eventually provide real computed evidence. | Requires instrumentation, algorithms, tests and possibly data; not authorized. | Source, tests, research engine, data, validation | Not recommended for current slice | Use for `FU-CR161-001`..`005` after approval. |
| Keep current slots only | No new model. | Leaves missing evidence ambiguous and risks overclaim. | Admission semantics, release wording | Rejected | N/A. |

### AGA-CR161-002: `typed_unavailable` Claim Ceiling

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Fail closed by claim tier | Prevents silent PASS and paper/production overclaim. | Blocks more historical artifacts. | Admission policy, QA, release wording | Recommended | Exploratory wording can be refined, but paper/production must stay blocked. |
| Warning-only `NEEDS_REVIEW` | Less disruptive. | Allows missing mandatory evidence to leak into stronger claims. | Risk acceptance, release wording | Rejected | Only possible with a new waiver policy and human risk acceptance. |
| Block all claims including exploratory | Maximally safe. | Too strict for honest research notes. | Product usability | Not recommended | Switch if users repeatedly misuse exploratory labels. |

### AGA-CR161-003: Integration Surface

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| CR151/CR154 refs and summaries | Preserves existing gate family; lowest policy drift. | Requires adapter discipline later. | Module boundaries, release wording, maintenance | Recommended | Switch only if CR151/154 are deprecated or cannot express required refs. |
| Parallel CR161 gate | Self-contained. | Duplicate status family and user confusion. | Architecture, testing, docs | Rejected | Only with a future deprecation plan for old gates. |
| Direct package-only fields | Simple consumer surface. | Mixes evidence availability with package display and hides gate ownership. | Admission package, maintainability | Not recommended | Use only as a downstream summary consumer, not the source of truth. |

### AGA-CR161-004: CR155 Negative Regression

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Blocked negative regression only | Honest about missing evidence; preserves strongest negative sample. | Does not produce positive proof. | Verification, historical evidence, release wording | Recommended | Reopen CP2 if CR155 evidence facts conflict. |
| Reconstruct historical C1/C2 | Would make CR155 look complete. | Risks fabricating facts. | Evidence integrity, audit risk | Rejected | N/A without independently verified source facts. |
| Omit CR155 | Avoids historical gap. | Loses valuable blocked regression sample. | Verification coverage | Rejected | Only if CR155 package is proven inconsistent. |

### AGA-CR161-005: Follow-up Split

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Five named follow-ups | Keeps deferred analytics auditable and separately routable. | Requires later governance. | Backlog, route planning, CP8 tracking | Recommended | Promote individual FU by user approval. |
| One large follow-up | Less bookkeeping. | Hides distinct risks and dependencies. | Planning, implementation risk | Rejected | N/A. |
| Implement all now | Completes more capability. | Violates current authorization and scope. | Code, data, runtime, validation | Rejected for current slice | Requires explicit route change. |

## Decision Items for CP3 Human Gate

| Decision ID | Type | Recommended Option | Alternatives | Impact / Risk | Rollback / Switch |
|---|---|---|---|---|---|
| `DQ-CP3-CR161-001` | architecture | Contract-first evidence availability overlay. | Compute-first implementation; documentation-only; parallel gate. | Keeps current slice design-only and truthful. | Switch only after explicit implementation authorization. |
| `DQ-CP3-CR161-002` | architecture | `typed_unavailable` fails closed by claim tier. | Warning-only; binary block; waiver path. | Prevents overclaim. | Relax only exploratory wording; keep higher-tier blocks. |
| `DQ-CP3-CR161-003` | architecture | Integrate via CR151/CR154 refs/summaries. | Parallel CR161 gate; package-only fields. | Preserves existing gate family. | Add adapters or reopen CP3 if CR151/154 cannot support refs. |
| `DQ-CP3-CR161-004` | verification | CR155 negative regression only. | Reconstruct history; omit CR155; remediation. | Prevents fabricated historical proof. | Reopen CP2 if facts conflict. |
| `DQ-CP3-CR161-005` | follow_up_tracking | Accept `FU-CR161-001`..`005` as deferred candidates. | Implement all now; collapse follow-ups. | Keeps computation work outside current scope. | Promote per follow-up CR. |
| `DQ-CP3-CR161-006` | security | No new implementation/runtime/data/credential/external/publish authorization. | Separate authorization route. | Prevents side effects. | Stop and request authorization if needed. |

## Deferred Ideas

| ID | Idea | Deferred Reason | Trigger |
|---|---|---|---|
| `FU-CR161-001` | Research-engine trial lineage instrumentation. | Requires source/test implementation and research-engine changes. | Need future `ExperimentFamilyManifest=present`. |
| `FU-CR161-002` | Multiple testing / PBO / DSR computable evidence implementation. | Requires computation logic and input contracts. | Need `MultipleTestingEvidence`, `DataSnoopingEvidence` or `OverfitRiskEvidence=present`. |
| `FU-CR161-003` | Walk-forward fold-level metrics and OOS evidence computation. | Requires split/fold metric producers. | Need `WalkForwardEvidence=present`. |
| `FU-CR161-004` | Economic cost / impact approximation implementation. | Requires cost/impact model boundary; real TCA remains separate. | Need `EconomicCostEvidence=present`. |
| `FU-CR161-005` | Capacity / liquidity sizing and alpha decay implementation. | Requires capacity and liquidity model. | Need `CapacityLiquidityEvidence=present`. |

## Not Authorized in This Discussion Outcome

- Source or test implementation.
- Research-engine instrumentation.
- Credential, secret or `.env` read.
- Real lake read or write.
- NAS read, write, sync or metadata normalization.
- Provider fetch.
- QMT/MiniQMT/xtquant/gateway runtime.
- Simulation, paper, live or trading runtime.
- Broker operation.
- Catalog/store/registry/model/prediction write.
- External framework clone, install or run.
- Git remote write.
- Publish or true release execution.

## Ready State

The HLD and ADR can be submitted to host-orchestrator for CP3 human gate preparation. This log does not approve CP3; it only records solution-design inputs and selected architecture recommendations.
