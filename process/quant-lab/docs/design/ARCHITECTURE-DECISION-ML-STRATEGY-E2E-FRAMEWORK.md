---
status: "cp3-review"
version: "0.1"
change: "CR-152"
title: "ADR: ML Strategy E2E Framework"
created_at: "2026-07-02T10:07:16+08:00"
owner: "host-orchestrator"
---

# ADR: ML Strategy E2E Framework

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-02 | host-orchestrator | 新增 CR152 CP3 ADR：ML-specific gate + adapter、metadata-only artifact boundary、extend existing contracts、fixture-only validation。 |

## ADR-CR152-001: ML-Specific Admission Gate with Shared Status Adapter

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | CR151 has four-state statistical gate semantics. ML first wave needs PIT, label, CV and artifact evidence that does not fit CR151 report schema. |
| Decision | Define an ML-specific admission gate summary and adapter that maps to `PASS / FAIL / NEEDS_REVIEW / BLOCKED` and admission package linkage. |
| Alternatives | A. Reuse CR151 gate directly; B. define fully separate ML status model. |
| Pros | Keeps ML evidence clear while preserving shared governance semantics. |
| Cons | Adds one adapter contract. |
| Impact | CP4 Story split must include ML gate contract and adapter tests. |
| Rollback / Switch | If CP5 proves ML evidence can be represented without loss in an existing object, adapter may become a thin mapping helper, but four-state semantics must remain. |

## ADR-CR152-002: Metadata Contract Is Not Model Registry Write

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | First wave includes training snapshot / model artifact metadata. User explicitly warned that this must not become model registry write. |
| Decision | CR152 may define metadata-only fields and validators for `TrainingSnapshotSpec` / `ModelArtifactRef` linkage, but must not create registry writers, publish methods, catalog pointer mutation or store writes. |
| Alternatives | A. Add local registry writer; B. integrate external registry framework. |
| Pros | Preserves static/fixture-only authorization and keeps CP6 verifiable without side effects. |
| Cons | Cannot claim model registry readiness or real artifact publication. |
| Impact | CP5/CP6 must treat verbs like write/publish/register/promote/upload/sync/mutate/set_current as out of scope for model/store/catalog state. |
| Rollback / Switch | Registry writes require a later storage/runtime authorization gate or formal CR. |

## ADR-CR152-003: Extend Existing Contracts, Do Not Rebuild Parallel ML Contracts

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | Existing `ResearchDatasetSpec`, `TrainingSnapshotSpec`, `ModelArtifactRef`, `StrategyAdmissionStatisticalGate` and `StrategyAdmissionPackage` already cover adjacent semantics. |
| Decision | CR152 first wave must extend or compose existing anchors. A narrow companion object is allowed only when it references the anchor object and does not replace it. |
| Alternatives | A. New `ml_contracts.py` family; B. documentation-only. |
| Pros | Satisfies ML-1 contract convergence and reduces future migration burden. |
| Cons | Requires careful delta mapping in CP5. |
| Impact | CP4/CP5 must include a contract delta table in Story design evidence. |
| Rollback / Switch | If a source anchor is found defective, fix or wrap the anchor; do not fork semantics silently. |

## ADR-CR152-004: Deterministic Fixture-Only Validation Boundary

| Field | Decision |
|---|---|
| Status | Recommended for CP3 approval |
| Context | CR152 must prove first-wave semantics without connecting real lake/NAS/provider/runtime or training a real model. |
| Decision | Use deterministic local fixtures with explicit PIT, leakage, purged/embargo and artifact metadata cases; all forbidden operation counters must remain zero. |
| Alternatives | A. Tiny schema-only fixtures; B. real lake validation. |
| Pros | Provides meaningful static verification while respecting authorization. |
| Cons | Does not prove real model performance or production readiness. |
| Impact | CP5 test plan must define fixture size, fields, positive cases and negative cases. |
| Rollback / Switch | Real data validation requires a later runtime/data authorization gate. |

## Not Authorized

- source implementation before CP5 approval
- real model training or real data validation
- real lake read/write
- NAS read/write/sync/restore/chmod/chgrp/metadata normalization
- provider fetch
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/paper/live/trading runtime
- broker read/write/submit/cancel/account query
- credential or `.env` read
- external framework clone/install/run
- Git remote write
- catalog pointer mutation
- feature store, label store, model registry or prediction store writes

## Downstream Obligations

| Stage | Obligation |
|---|---|
| CP4 | Split Stories only after CP3 approval; preserve S01..S05 candidate grouping or document changes. |
| CP5 | Each full LLD must include anchor contract mapping, fixture cases, no-write/no-runtime guardrails and design-delta obligations. |
| CP6 | Implementation must use local/static/fixture tests only; any registry/store/publish writer is a scope violation. |
| CP7 | Verification wording must state fixture semantics only, not real model performance. |
| CP8 | Release readiness must carry any residual process caveats and no-runtime/no-registry limitations. |
