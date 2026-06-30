# CR139-S16 LLD - Training Snapshot Cutoff

## 1. Story Scope

S16 defines a deterministic training snapshot/cutoff contract for ML experiments. It consumes S10 lake/as-of input contract and S12 manifest closure, then validates that training input references a published snapshot and fixed training cutoff. It does not perform real lake reads or modify `readers.py` runtime behavior.

## 2. Inputs

- `experiments/lake_input_contract.py`
- New `experiments/training_snapshot_contract.py`
- S12 `ExperimentManifestClosure`

## 3. Output Contract

Add:

- `TrainingSnapshotSpec`
- `TrainingSnapshotValidation`
- `build_training_snapshot_spec(...)`
- `validate_training_snapshot_cutoff(...)`

The contract records:

- snapshot id/path/content hash
- `published_only=true`
- `as_of`
- `training_cutoff`
- split policy id
- feature schema hash
- lineage refs
- zero operation counters

## 4. Validation Rules

- Training snapshot must reference published snapshot id/path/content hash.
- `training_cutoff` must be present and `<= as_of`.
- `latest`, `current`, or mutable pointer style snapshot refs are blocked.
- Feature schema hash and split policy id are required.
- Permission counters must be zero.

## 5. File Ownership

- `experiments/training_snapshot_contract.py`
- `tests/test_cr139_training_snapshot_cutoff.py`

S16 does not modify `market_data/readers.py` because S10 already established the lake/as-of reader boundary; changing core reader logic is unnecessary for this fixture/static contract.

## 6. Test Plan

- Valid published snapshot and cutoff passes.
- Missing published ref blocks.
- `latest/current` mutable refs block.
- Cutoff after as_of blocks.
- Nonzero operation counter blocks.

## 7. Dependencies

- S04 published pointer read selector contract.
- S10 lake/as-of adapter.
- S12 manifest closure.

## 8. No-Real-Operation Boundary

No provider, lake/catalog/manifest/pointer, NAS, runtime, QMT, simulation/live, credential, dependency, or Git remote operation.

## 9. Acceptance Mapping

- REQ-218/V2: ML training reads a published snapshot reference with fixed, reproducible training cutoff.

## 10. CP5 Decision

Proceed to CP6 fixture/static implementation. No human gate required.
