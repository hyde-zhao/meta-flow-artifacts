---
status: "cp3-review"
version: "0.1"
change: "CR-152"
title: "ML Strategy E2E Framework HLD"
created_at: "2026-07-02T10:07:16+08:00"
owner: "host-orchestrator"
scope_boundary: "local/static/fixture-only; metadata contracts only; no model registry write, no real training, no real data validation"
---

# HLD: ML Strategy E2E Framework

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-02 | host-orchestrator | CR152 CP3 初稿：定义 ML first-wave 架构、现有 contract 扩展策略、metadata vs registry write 边界、ML gate adapter、fixture-only 验证边界。 |

## 1. Problem

CR151 已完成多因子 `StrategyAdmissionStatisticalGate`，但 ML 策略仍缺少端到端的本地契约框架：PIT feature matrix、label policy、purged + embargo CV、training snapshot / model artifact metadata、prediction artifact 和 ML admission gate 尚未形成可验证链路。

CR152 的 CP3 目标不是实现真实模型训练，也不是连接 lake/NAS/provider/runtime/model registry，而是确认一个可进入 CP4/CP5 的架构：后续 first-wave Story 可以用 local/static/fixture 证明 ML 策略 E2E 契约语义，不产生真实数据、真实模型或真实发布副作用。

## 2. Goals and Success Criteria

| ID | Goal | Measurable Success Criteria |
|---|---|---|
| G1 | 收敛 ML first-wave 架构 | HLD 覆盖 6 项 first-wave prerequisites，并为每项给出模块边界、输入、输出和 fail path。 |
| G2 | 复用现有契约 | 至少 5 个现有对象被列为 extension anchors：`ResearchDatasetSpec`、`TrainingSnapshotSpec`、`ModelArtifactRef`、`StrategyAdmissionStatisticalGate`、`StrategyAdmissionPackage`。 |
| G3 | 固化 no-registry-write 边界 | HLD/ADR/CP3 checkpoint 至少 3 处明确 model registry write/publish/catalog pointer mutation 不授权。 |
| G4 | 支持 CP5 设计证据 | CP3 输出至少 4 个 CP3 DQ、4 个 ADR、1 个 contract delta table 和 1 个 failure-path table。 |
| G5 | Fixture-only 可验证 | Fixture contract 至少包含 10 个必需字段/概念：entity、date、decision_time、feature_available_at、label_available_at、label_window、fold boundary、purge window、embargo gap、prediction_timestamp。 |

## 3. Non-Goals

| Non-goal | Boundary |
|---|---|
| Real model training | 不运行 fit/train/tune，不产出真实模型性能。 |
| Real data validation | 不读真实 lake、NAS、provider、catalog current truth 或生产数据快照。 |
| Model registry write | 不写 registry，不 publish model，不 mutate catalog pointer，不写 feature/label/model/prediction store。 |
| Runtime / trading | 不启动 QMT、MiniQMT、xtquant、gateway、simulation、paper、live 或 broker 操作。 |
| External ML framework | 不 clone/install/run qlib、mlfinlab、sklearn pipeline 或其他外部框架。 |
| Full ML methodology | triple-barrier 算法、meta-label training、feature importance、sample uniqueness weighting、drift/retrain、fractional differentiation、class imbalance 和 MinBTL 均为 later wave。 |

## 4. Candidate Architectures

| Option | Description | Pros | Cons | Decision |
|---|---|---|---|---|
| A. Contract extension + ML-specific gate + adapter | 在现有研究契约上增加 ML companion objects，ML gate 独立表达 ML 证据，通过 adapter 复用 CR151 四态 status 和 admission package linkage。 | 契约收敛、证据清晰、可 fixture-only 测试、可后续扩展 triple-barrier 和 registry。 | 需要新增 ML companion contracts 和 adapter。 | Recommended |
| B. Reuse CR151 gate directly | 把 ML 证据压缩进 `StrategyAdmissionStatisticalGate` report refs 和 blocked reasons。 | 新模块少。 | ML PIT/label/CV/artifact 语义不清，CR151 gate 被误用为 ML gate。 | Rejected |
| C. Parallel ML framework | 新建完整 ML contract family，不依赖现有 research/admission contracts。 | ML 命名清晰。 | 违反 ML-1 contract convergence，重复 `ResearchDatasetSpec`/`TrainingSnapshotSpec`/`ModelArtifactRef`。 | Rejected |

## 5. Recommended Architecture

CR152 采用 Option A：metadata-only ML contract extensions + ML-specific admission gate + adapter to shared admission semantics。

```text
Local deterministic fixture
        |
        v
ResearchDatasetSpec extension anchor
        |
        v
PITFeatureMatrixSpec --> LabelPolicySpec --> PurgedEmbargoCVSpec
        |                    |                    |
        v                    v                    v
FeatureMatrixAudit    LabelLeakageAudit     SplitAudit
        \                    |                    /
         \                   v                   /
          ----> TrainingSnapshotSpec extension ----
                              |
                              v
             ModelArtifactRef metadata extension
                              |
                              v
              PredictionArtifactSpec / audit
                              |
                              v
                 MLAdmissionGateSummary
                              |
                              v
   CR151 four-state status adapter + StrategyAdmissionPackage linkage
```

## 6. Contract Delta Table

| Existing Anchor | Current Role | CR152 Extension / Companion | Boundary |
|---|---|---|---|
| `ResearchDatasetSpec` in `engine/research_production_contracts.py` | Research dataset metadata, date window, features, labels, leakage policy, source-of-truth guards. | Add/compose `PITFeatureMatrixSpec` with feature availability fields, primary keys, feature schema hash and fixture snapshot refs. | No lake read/write; no feature store publish. |
| `LeakagePolicy` in `engine/research_production_contracts.py` | Decision time, feature availability and label availability policy. | Add/compose `MLLabelPolicySpec` with `label_method` enum `fixed_window | triple_barrier | meta_label` and optional `method_spec`. | First wave validates fixed-window and reserved slots; no triple-barrier algorithm. |
| `ExperimentSplitPolicy` in `engine/research_manifest.py` | Train/validation/test split metadata with embargo days and label horizon. | Add/compose `PurgedEmbargoCVSpec` and `PurgedEmbargoSplitAudit` with fold boundaries, purge windows, embargo gaps and overlap checks. | No real training; no runtime CV execution. |
| `TrainingSnapshotSpec` in `engine/training_snapshot_contract.py` | Published snapshot metadata and cutoff validation. | Extend metadata linkage with ML split policy id, PIT feature schema hash and label policy fingerprint. | Metadata only; no snapshot publish or catalog pointer write. |
| `ModelArtifactRef` in `engine/research_manifest.py` | Model artifact id/path/hash and dataset snapshot hash. | Add/compose `MLModelArtifactMetadata` with training snapshot ref, hyperparameter/config hash, feature schema hash and limitations. | Artifact path/ref is metadata; not a registry write, not model storage write. |
| `StrategyAdmissionStatisticalGate` in `engine/strategy_admission_statistical_gate.py` | Four-state statistical gate and evidence issues. | Reuse status values through adapter; do not reuse CR151 report schema as ML report schema. | Adapter maps semantics only. |
| `StrategyAdmissionPackage` in `engine/strategy_admission_package.py` | Offline admission evidence package with runtime claim blockers. | Add/compose ML gate summary fields: `gate_present`, `gate_required`, `gate_status`, `gate_ref`, `blocked_reasons`. | Does not imply runtime, simulation, live or broker readiness. |

## 7. Module Responsibilities

| Module / Area | Responsibility | Inputs | Outputs | Failure Path |
|---|---|---|---|---|
| `engine.research_production_contracts` | Own ML dataset/PIT companion contracts near `ResearchDatasetSpec`. | Static fixture metadata, feature/label refs. | `PITFeatureMatrixSpec`, `MLLabelPolicySpec`, audits. | Missing PIT fields or mutable refs -> `BLOCKED`. |
| `engine.training_snapshot_contract` | Remain training snapshot cutoff anchor; accept ML metadata refs by composition/extension. | Training snapshot metadata, split policy id, feature schema hash. | Snapshot validation result. | Cutoff after as-of or nonzero forbidden counters -> `BLOCKED`. |
| `engine.research_manifest` | Remain artifact/manifest anchor; host or link `ModelArtifactRef` metadata extension. | Model artifact metadata refs. | `MLModelArtifactMetadata` / existing model ref linkage. | Missing artifact hash/dataset hash -> `BLOCKED`. |
| New ML admission companion module | Evaluate ML first-wave evidence into ML gate summary. | PIT audit, label audit, split audit, training/model/prediction metadata. | `MLAdmissionGateSummary`, issues, operation counts. | Mandatory evidence missing -> `BLOCKED`; threshold/semantic failure -> `FAIL`; incomplete but reviewable evidence -> `NEEDS_REVIEW`. |
| Admission adapter | Map ML gate status to shared four-state semantics and package linkage. | `MLAdmissionGateSummary`. | Package summary fields and blocked reasons. | Unknown status or adapter mismatch -> `BLOCKED`. |

## 8. Metadata vs Registry Write Boundary

| In Scope Metadata Contract | Explicitly Out of Scope Registry/Store Write |
|---|---|
| `model_id`, `artifact_ref`, `artifact_hash`, `dataset_snapshot_hash`, `training_snapshot_ref`, `feature_schema_hash`, `config_hash`, `lineage_refs`, `limitations`, `operation_counts`. | Creating registry entries, updating registry aliases, publishing model versions, writing feature/label/model/prediction stores, mutating catalog pointers, uploading artifact binaries, marking production/champion models. |
| Static validation that required fields are present and immutable-looking. | Any API, filesystem or network operation that writes a model, registry record, catalog pointer or production truth. |
| Fixture claims limited to contract semantics and no-real-op counters. | Claims of real model performance, production readiness, paper/live readiness or trading readiness. |

Implementation guardrail for CP5/CP6: any function whose primary action is `write`, `publish`, `register`, `promote`, `upload`, `sync`, `persist`, `mutate` or `set_current` for model/feature/label/prediction/catalog state is outside CR152 first wave.

## 9. Key Flows

### Flow 1: Fixture Contract Assembly

1. Build deterministic fixture rows with entity/date keys, decision time, feature availability time, label availability time, label window and prediction timestamp.
2. Build `ResearchDatasetSpec`-anchored PIT feature matrix metadata.
3. Build `MLLabelPolicySpec` with `fixed_window` active and `triple_barrier` / `meta_label` slots reserved but inactive.
4. Build `PurgedEmbargoCVSpec` and split audit.
5. Build training snapshot and model artifact metadata refs.
6. Build prediction artifact metadata.
7. Evaluate ML admission gate and adapter summary.

### Flow 2: Fail-Closed Evidence

| Failure | Expected Status | Reason |
|---|---|---|
| Missing `feature_available_at` | `BLOCKED` | PIT semantics cannot be proven. |
| Feature availability after decision time | `BLOCKED` | Leakage guard violated. |
| Label availability before decision time | `BLOCKED` | Forward label policy invalid. |
| Purge/embargo overlap detected | `FAIL` or `BLOCKED` | Split audit fails; if metadata absent then blocked. |
| Model artifact hash missing | `BLOCKED` | Artifact metadata cannot be linked. |
| Registry write counter nonzero | `BLOCKED` | Not authorized operation attempted. |
| `triple_barrier` selected as active method in first wave | `NEEDS_REVIEW` or `BLOCKED` | Slot is reserved but algorithm not implemented; CP5 must decide exact enforcement. |

## 10. Use Case Traceability

| Use Case / Gap | Architecture Coverage | Verification Mode |
|---|---|---|
| UC-59 ML strategy E2E | Full first-wave metadata chain from PIT feature matrix to ML admission gate. | Unit tests + fixture validation + CP7 evidence. |
| ML-GAP-1 leakage | PIT feature availability and label policy audits. | Deterministic leakage fixtures. |
| ML-GAP-2 purged + embargo CV | `PurgedEmbargoCVSpec` and split audit. | Fold overlap and embargo gap fixtures. |
| ML-GAP-3 triple-barrier / meta-labeling | Reserved label method enum/spec slots. | Static contract validation only in first wave. |
| ML-GAP-12 training snapshot / artifact lineage | `TrainingSnapshotSpec` and `ModelArtifactRef` extension/composition. | Metadata field and hash/ref validation. |
| CR151 statistical gate relation | Adapter reuses four-state status and admission package linkage. | Adapter mapping tests. |

## 11. Fixture Data Contract

| Requirement | Minimum |
|---|---|
| Entity keys | At least one stable entity id column and one date/time column. |
| Time span | At least 5 chronological blocks so train, validation, test, purge and embargo gaps are distinguishable. |
| Required timestamps | `decision_time`, `feature_available_at`, `label_available_at`, `prediction_timestamp`. |
| Label fields | `label_window_start`, `label_window_end`, `label_horizon_days`, `label_method`. |
| CV fields | fold id, train start/end, validation start/end, test start/end, purge window, embargo gap. |
| PIT cases | At least 1 passing row and 1 leakage row where feature availability is after decision time. |
| Split cases | At least 1 passing fold set and 1 overlap/embargo violation case. |
| Operation counters | All forbidden operation counters must be zero. |
| Claim limit | Fixture proves contract semantics only; it does not prove model performance. |

## 12. Security and Authorization Boundary

CR152 CP3 approval does not authorize:

- source implementation before CP5 approval
- real model training or hyperparameter search
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

## 13. Risks and Mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| Metadata contract is misread as model registry capability | high | HLD/ADR and tests must name registry write as forbidden; CP5 story design must include no-write checks. |
| Parallel ML contracts duplicate existing anchors | high | CP5 must include contract delta mapping; no new object may replace `ResearchDatasetSpec`, `TrainingSnapshotSpec` or `ModelArtifactRef`. |
| Fixture tests are too small to prove purged + embargo CV | medium | CP5 test plan must require minimum fixture schema/time span and negative cases. |
| `triple_barrier` slot is mistaken for implemented algorithm | medium | Label policy must expose explicit method support state; CP8 wording must say slot reserved only. |
| ML gate PASS is misread as live readiness | high | Admission package summary must retain runtime claim blockers and no-real-op limitations. |

## 14. Tentative Story Groups After CP3 Approval

Story decomposition is not authorized until CP3 is approved. If approved, CP4 may split along these candidate groups:

| Candidate Story | Scope | Design Evidence Level |
|---|---|---|
| CR152-S01 | PIT feature matrix and label policy contracts with leakage guard. | full-lld |
| CR152-S02 | Purged + embargo CV split policy and fixture data contract. | full-lld |
| CR152-S03 | Training snapshot / model artifact / prediction artifact metadata linkage. | full-lld |
| CR152-S04 | ML admission gate and CR151 status/admission package adapter. | full-lld |
| CR152-S05 | CP7 fixture verification and release evidence wiring. | technical-note |

## 15. Gotchas

1. Do not treat `ModelArtifactRef` as proof that a model artifact was written or registered.
2. Do not create a new ML dataset contract that replaces `ResearchDatasetSpec`; add companion fields or validators.
3. Do not make `triple_barrier` active in first-wave tests unless a later decision explicitly expands scope.
4. Do not let ML gate `PASS` bypass CR151/CR154 admission governance or runtime authorization.
5. Do not use real lake/NAS/provider fixtures to make local tests look more realistic.

## 16. Open Questions

| ID | Question | Status | Resolution |
|---|---|---|---|
| O-CR152-CP3-001 | Should CR152 include model registry writes? | RESOLVED | No. Metadata-only contract is in scope; registry writes are out of scope. |
| O-CR152-CP3-002 | Should TrainingSnapshotSpec / ModelArtifactRef be rebuilt? | RESOLVED | No. Existing anchors must be extended/composed. |
| O-CR152-CP3-003 | Should CP3 discussion checkpoint be written? | RESOLVED | Yes. Architecture Gray Areas were explored; discussion log and checkpoint are included. |

## 17. Self Review Checklist

| Rule | Result |
|---|---|
| Internal consistency | PASS: HLD/ADR/CP3 DQ all use Option A and metadata-only boundary. |
| Quantified goals | PASS: §2 includes measurable counts and field requirements. |
| Integration contracts | PASS: §6 and §7 define anchors, inputs, outputs and failure paths. |
| Adjacent boundaries | PASS: §3, §8 and §12 separate metadata from registry/runtime. |
| Failure paths | PASS: §9 includes fail-closed table. |
| Rollback / switch | PASS: advisor tables include switch conditions; CP3 Decision Brief includes rollback. |
| Theory source | PASS: purged + embargo CV, PIT leakage and admission gate status are traced to roadmap/remediation/E2E review and CR151. |
| Open question status | PASS: §16 marks all CP3 open questions resolved. |
| Revision record | PASS: §0 includes revision row. |
