---
discussion_id: "CP3-CR152-HLD-DISCUSSION"
cr_id: "CR-152"
phase: "solution-design"
status: "ready-for-cp3-review"
owner: "host-orchestrator"
created_at: "2026-07-02T10:07:16+08:00"
authorization_boundary: "design-only; no source implementation, no Story decomposition, no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/model registry/catalog pointer"
---

# CP3 CR152 HLD Discussion Log

## Purpose

This log records the Architecture Gray Areas used before drafting the CR152 ML Strategy E2E Framework HLD. It is an audit and recovery artifact. The downstream authority after CP3 approval is the HLD, ADR, Feature Design Matrix, Story cards and CP5 design evidence.

## Inputs Scanned

| Source | Path | Result |
|---|---|---|
| CP2 approval | `process/checkpoints/CP2-CR152-ML-STRATEGY-E2E-SCOPE.md` | CR152 first-wave scope, security boundary, ML gate relation, statistical gate opt-in, label extensibility and fixture contract were approved. |
| CP3 context capsule | `process/context/CP3-CR152-ML-STRATEGY-E2E-HLD-CONTEXT.yaml` | Three CP3 focus items must be addressed: metadata vs registry boundary, extension over rebuild, CP3 gray-area tracking. |
| Roadmap / remediation plan | `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md`, `docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md` | CR152 first wave contains six prerequisites and must remain local/static/fixture-only. |
| E2E review | `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` | ML gaps remain valid, while the old claim that ResearchDatasetSpec / BacktestRunSpec files are absent is stale. |
| Source contracts | `engine/research_production_contracts.py`, `engine/training_snapshot_contract.py`, `engine/research_manifest.py`, `engine/strategy_admission_statistical_gate.py`, `engine/strategy_admission_package.py` | Existing `ResearchDatasetSpec`, `TrainingSnapshotSpec`, `ModelArtifactRef`, four-state statistical status semantics and admission package linkage are extension anchors. |

## Architecture Gray Areas

| AGA ID | Key Question | Why It Changes Architecture | Impact Surface | Recommendation | Status |
|---|---|---|---|---|---|
| AGA-CR152-01 | Should ML admission reuse CR151 statistical gate directly or define an ML-specific gate with adapter linkage? | This decides whether ML-specific PIT / label / CV / artifact evidence is modeled as first-class ML evidence or squeezed into multifactor statistical report fields. | module boundary, status semantics, admission package linkage, tests | Define an ML-specific admission gate and adapter that reuses CR151 four-state semantics and admission package linkage. | selected |
| AGA-CR152-02 | Where is the boundary between model artifact metadata and model registry write? | Confusing metadata contracts with registry publish would violate CP2 no-write authorization and make CP5/CP6 implementation unsafe. | contract shape, forbidden paths, tests, security | First wave may define metadata-only `ModelArtifactRef`/training snapshot extensions; model registry write/publish APIs are not in scope. | selected |
| AGA-CR152-03 | Should CR152 extend existing contracts or create a parallel ML contract family? | Parallel contracts would undermine ML-1 contract convergence and duplicate existing `ResearchDatasetSpec`, `TrainingSnapshotSpec` and `ModelArtifactRef` semantics. | source module ownership, migration risk, future maintenance | Extend or compose existing contracts in `engine/research_production_contracts.py`, `engine/training_snapshot_contract.py` and `engine/research_manifest.py`; do not rebuild. | selected |
| AGA-CR152-04 | How should label policy handle triple-barrier / meta-labeling without implementing algorithms now? | Omitting the extension slots would force later ML-3 work into incompatible contract changes or parallel implementation. | label policy schema, future ML-3, fixtures | Reserve `fixed_window`, `triple_barrier` and `meta_label` enum/spec slots, but first wave only validates static contract semantics. | selected |
| AGA-CR152-05 | What must fixture data prove for purged + embargo CV? | If fixtures are too small or lack timestamps, the CP6 implementation can pass empty-shell tests while missing the core leakage guard. | fixture schema, split validator, test plan | Require deterministic fixture schema with enough time span and explicit PIT/leakage/overlap cases; no real data validation. | selected |

## Advisor Tables

### AGA-CR152-01 ML Gate Relation

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| ML-specific gate with CR151 status adapter | Preserves ML-specific evidence shape; reuses proven `PASS / FAIL / NEEDS_REVIEW / BLOCKED` semantics; keeps admission package linkage consistent. | Adds one adapter and ML gate contract. | gate module, admission package summary, tests | Recommended. | Switch only if CP5 proves ML gate evidence is identical to CR151 reports, which is unlikely. |
| Reuse `StrategyAdmissionStatisticalGate` directly | Minimal new gate code. | Forces PIT feature, label, CV and artifact checks into non-ML statistical fields; reduces audit clarity. | maintainability, traceability | Not recommended. | N/A for first wave. |
| Fully separate ML status model | Maximum flexibility. | Duplicates status semantics and complicates cross-strategy admission governance. | governance, CP8 wording | Not recommended. | Consider only if CR154 defines a new shared admission status protocol. |

### AGA-CR152-02 Metadata Contract vs Registry Write

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Metadata-only artifact contract | Matches CP2; deterministic fixture tests; no publish or storage side effect. | Cannot claim model registry availability. | contracts, tests, security | Recommended. | Switch only through a later runtime/storage authorization gate. |
| Include local registry writer | Would exercise an end-to-end write path. | Violates CR152 no-write scope and risks catalog/pointer mutation confusion. | security, state, release | Not authorized. | Future CR only. |
| Integrate external registry framework | Reuses industry tooling. | Adds dependency, install and runtime surface; violates external framework boundary. | dependency, runtime | Not authorized. | Future Spike with explicit dependency authorization. |

### AGA-CR152-03 Contract Extension Strategy

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Extend/compose existing contracts | Aligns ML-1 convergence; reduces duplicate validation; keeps existing manifests and admission packages relevant. | Requires careful naming and delta mapping. | source modules, HLD, CP5 design evidence | Recommended. | If an existing contract proves incompatible, add a narrow companion object with explicit ref to the anchor object. |
| New `ml_contracts.py` parallel family | Clear local namespace. | Duplicates `ResearchDatasetSpec`, `TrainingSnapshotSpec`, `ModelArtifactRef`; future bridging burden. | maintenance, migration | Not recommended. | N/A. |
| Documentation-only design | No code churn. | Does not complete first-wave framework foundation. | value, verification | Not recommended. | Use only if CP3 is rejected. |

### AGA-CR152-04 Label Extensibility

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Reserve enum + spec slot | Avoids future contract break; keeps triple-barrier/meta-labeling in same label policy family. | Requires explicit not-implemented semantics for non-fixed methods. | schema, tests, future ML-3 | Recommended. | Implement algorithms only in later ML-3 scope. |
| Fixed-window only | Fastest first-wave implementation. | Freezes a serious ML-GAP into the contract and encourages parallel future work. | future compatibility | Not recommended. | N/A. |
| Implement triple-barrier now | Completes more methodology. | Expands CP6 beyond first-wave contract foundation and may require real data assumptions. | scope, tests | Not authorized for CR152 first wave. | Future CR/wave. |

### AGA-CR152-05 Fixture Contract

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Deterministic local fixture contract | Proves split/purge/embargo/leakage semantics without real data. | Fixture performance claims are not real model performance. | tests, CP7 wording | Recommended. | Switch only if later user authorizes real data validation. |
| Tiny schema-only fixture | Minimal effort. | Cannot prove purged + embargo CV behavior. | test quality | Not recommended. | N/A. |
| Real lake validation fixture | More realistic. | Violates no real lake and no provider boundary. | security, runtime | Not authorized. | Future runtime/data CR only. |

## Adopted Inputs

| Input | HLD Effect |
|---|---|
| CP2 DQ-CP2-CR152-ML-GATE-RELATION | HLD defines ML-specific gate plus CR151 status/admission linkage adapter. |
| CP2 DQ-CP2-CR152-STAT-GATE-OPT-IN | HLD requires admission package summary fields `gate_present`, `gate_required`, `gate_status`; default enforcement remains opt-in. |
| User CP3 focus item 1 | HLD and ADR include explicit metadata contract vs registry write boundary and forbidden path list. |
| User CP3 focus item 2 | HLD includes a contract delta table anchored to existing `ResearchDatasetSpec`, `TrainingSnapshotSpec` and `ModelArtifactRef`. |
| User CP3 focus item 3 | This discussion log and `process/checks/CP3-CR152-DISCUSSION-CHECKPOINT.json` are created before CP3 gate launch. |

## Deferred Architecture Ideas

| ID | Idea | Deferred Reason | Trigger |
|---|---|---|---|
| DAI-CR152-01 | Triple-barrier labeling algorithm | CP2 approved only extension slot reservation, not algorithm implementation. | Later ML-3 wave or explicit scope expansion. |
| DAI-CR152-02 | Meta-label training | Requires real training semantics or broader ML workflow decisions. | Future ML wave. |
| DAI-CR152-03 | Feature importance reports (MDI/MDA/SHAP) | High-priority later wave, not first-wave contract foundation. | CR152 follow-up or CR154. |
| DAI-CR152-04 | Sample uniqueness weighting | Methodology item deferred by remediation plan. | Future ML wave. |
| DAI-CR152-05 | Drift / retrain trigger | Requires monitoring/runtime assumptions. | Production monitoring CR. |
| DAI-CR152-06 | Model registry writer or external registry integration | Storage/publish side effect not authorized. | Future storage/runtime authorization gate. |
