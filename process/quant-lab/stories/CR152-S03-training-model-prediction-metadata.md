---
story_id: "CR152-S03-training-model-prediction-metadata"
change_id: "CR-152"
title: "Training, model and prediction metadata"
status: "ready-for-verification"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
depends_on:
  - "CR152-S01-pit-feature-label-contracts"
  - "CR152-S02-purged-embargo-cv-fixture-contract"
implementation_allowed: true
authorization_boundary: "metadata only; no model registry write, real training, real data validation, lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/catalog pointer/Git remote"
---

# CR152-S03 Training, Model and Prediction Metadata

## Goal

Define training snapshot, model artifact and prediction artifact metadata linkage by extending or composing existing anchors.

## Scope

- `TrainingSnapshotSpec` companion linkage for ML split and feature/label fingerprints.
- `ModelArtifactRef` metadata extension or companion object.
- Prediction artifact metadata contract.
- No-registry-write and no-store-write operation counters.

## Acceptance Criteria

- Contract delta table maps every new field to an existing anchor or justified companion object.
- Model artifact metadata remains ref/hash/linkage only.
- Prediction metadata does not write a prediction store.
- Forbidden operation counters include registry write, store write and catalog pointer mutation.
- No model training, artifact binary persistence, registry writer, publish, promote, upload or set_current function is introduced.

## File Ownership

| File | Intent |
|---|---|
| `engine/training_snapshot_contract.py` | Training snapshot metadata linkage. |
| `engine/research_manifest.py` | Model artifact metadata companion linkage. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Metadata and no-registry-write fixtures. |

## Design Evidence Required For CP5

Full LLD covering contract delta mapping, field ownership, serialization, forbidden operation counters, prediction metadata semantics and rollback strategy.
