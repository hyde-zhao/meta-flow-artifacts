---
cr_id: "CR-152"
stage: "CP6"
status: "implemented"
created_by: "host-orchestrator-inline"
created_at: "2026-07-02T11:24:55+08:00"
validation_mode: "local-static-fixture"
runtime_authorization: "not_authorized"
---

# CR152 ML Strategy E2E Framework Implementation

## Scope

Implemented the CP5-approved local/static/fixture CR152 first-wave ML strategy E2E foundation:

- S01 PIT feature matrix contract and label policy / leakage guard contract.
- S02 purged + embargo CV policy contract and static validation.
- S03 training snapshot, model artifact and prediction artifact metadata companions.
- S04 ML-specific admission gate, four-state status semantics and admission package adapter.
- S05 fixture/static-only evidence and release wording boundary.

## Implemented Files

| File | Change | Story |
|---|---|---|
| `engine/research_production_contracts.py` | Added ML PIT feature matrix, label policy and purged/embargo CV contracts plus validators. | S01 / S02 |
| `engine/training_snapshot_contract.py` | Added `MLTrainingSnapshotMetadata` companion and validator. | S03 |
| `engine/research_manifest.py` | Added `MLModelArtifactMetadata` and `MLPredictionArtifactMetadata` companions and validators. | S03 |
| `engine/ml_strategy_admission_gate.py` | Added ML admission gate status, issue, evaluator and summary helper. | S04 |
| `engine/strategy_admission_package.py` | Added ML gate status mapping and package attachment helper without changing runtime flags. | S04 |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Added CR152 contract, metadata, gate and adapter tests. | S01-S05 |

## Design Contract Mapping

| CP5 Contract | Implementation Evidence |
|---|---|
| PIT feature matrix is metadata-only | `MLPITFeatureMatrixContract` stores field names, feature columns, lineage and zero operation counters only. |
| Label policy reserves future methods | `MLLabelPolicySpec` supports `fixed_window`; active `triple_barrier` / `meta_label` returns `ml_label_method_not_implemented` and aggregates to `BLOCKED`. |
| Purged + embargo CV is deterministic | `MLPurgedEmbargoCVPolicy` validates purge window, fold date order, overlap and embargo gap. |
| Metadata vs registry write boundary | S03 companions carry model/prediction metadata only; CR152 ML-only forbidden counters block real training, real data validation, model registry write, feature/label/model/prediction store write and catalog pointer mutation claims. |
| Extend existing anchors, do not rebuild | S03 extends `TrainingSnapshotSpec` and `ModelArtifactRef`; `engine/research_production_contracts.py` remained S01/S02 owned. |
| ML gate reuses four-state semantics | `MLAdmissionGateStatus` exposes `PASS / FAIL / NEEDS_REVIEW / BLOCKED`. |
| Admission package adapter preserves runtime flags | `attach_ml_gate_to_admission_package(...)` attaches `gate_present / gate_required / gate_status / gate_ref` to payload only and keeps `not_qmt_authorization`, `not_simulation_authorization`, `not_live_authorization` and `not_broker_order`. |

## Verification

```text
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_ml_strategy_e2e_contracts.py
5 passed in 0.47s

PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_research_production_contracts.py tests/research/test_strategy_admission_package.py tests/test_cr151_strategy_admission_statistical_gate.py tests/research/test_ml_strategy_e2e_contracts.py
34 passed in 0.64s

uv run --python 3.11 python -m py_compile engine/research_production_contracts.py engine/training_snapshot_contract.py engine/research_manifest.py engine/ml_strategy_admission_gate.py engine/strategy_admission_package.py tests/research/test_ml_strategy_e2e_contracts.py
passed
```

## Post-review Scope Hygiene

- Removed CR150 / CR151 test file renames from the CR152 CP6 worktree and evidence.
- Removed unrelated `tests/PROVENANCE.yaml` and guardrail script changes from the CR152 CP6 worktree and evidence.
- Full-suite test naming hygiene remains outside CR152 first-wave scope and should be handled by an independent hygiene follow-up if authorized.

## Boundary Check

| Boundary | Result |
|---|---|
| Real training / model fitting | 0 / not authorized |
| Real data validation | 0 / not authorized |
| Model registry/store/catalog write | 0 / not authorized |
| Real lake read/write | 0 / not authorized |
| NAS access/sync/write | 0 / not authorized |
| Provider fetch | 0 / not authorized |
| QMT/runtime/simulation/live/trading/broker | 0 / not authorized |
| Credential or `.env` read | 0 / not authorized |
| External framework install/run | 0 / not authorized |
| Git remote write | 0 / not authorized |

## Next Stage

Ready for CP7 local/static/fixture verification. CP7 should verify:

- Contract serialization and validator behavior.
- Active `triple_barrier` / `meta_label` first-wave `BLOCKED` enforcement.
- Purged/embargo CV fail-closed overlap and gap checks.
- Metadata-only / no-registry-write boundary.
- ML gate adapter status mapping and runtime flag preservation.
- S05 wording: contract semantics only, no real model performance claim.
