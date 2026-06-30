# CR139-S12 LLD - Experiment Manifest Closure

## 1. Story Scope

S12 closes experiment manifest lineage for CR139 ML/research workflows. It extends the existing CR030 research manifest contract with CR139 snapshot, as-of, split, feature artifact, model artifact, and lineage closure fields. It does not publish catalog entries or write lake data.

## 2. S29 Correctness Standard Reference

S12 validation adopts the S29 correctness standard from `process/stories/CR139-S29-pit-dedup-correctness-standard-LLD.md`:

- PIT standard: all decision inputs must be bounded by explicit `as_of` and must not admit future-available records.
- Dedup standard: a dataset snapshot must declare primary keys and duplicate policy, and validation must fail closed on duplicate key ambiguity.
- Fixture mode: correctness is proven with deterministic in-memory/tmp_path fixtures, not real lake or provider reads.

S12 manifest closure records the S29 standard reference in metadata and test assertions, so acceptance cannot drift from S29.

## 3. Inputs

- `engine/research_manifest.py`
- Existing CR030 `ExperimentManifest` and validation helpers
- S11 feature/label artifact refs
- Fixture/static tests only

## 4. Output Contract

Add CR139-specific dataclasses and helpers:

- `PublishedDatasetSnapshotRef`
- `ExperimentSplitPolicy`
- `ModelArtifactRef`
- `ExperimentManifestClosure`
- `build_experiment_manifest_closure(...)`
- `validate_experiment_manifest_closure(...)`

The closure must serialize with `to_dict()` and include:

- published dataset snapshot id/path/hash
- `as_of`
- split cutoff/embargo/label horizon
- feature/label artifact refs
- model artifact hash and dataset snapshot hash reference
- lineage refs and S29 standard ref
- zero permission counters

## 5. Validation Rules

Validation fails closed when:

- snapshot id/path/hash/as_of is missing
- split cutoff, embargo, or label horizon is missing or invalid
- model artifact hash is missing
- model artifact dataset snapshot hash does not match published snapshot hash
- feature/label artifact refs are missing
- S29 standard ref is missing
- forbidden truth sources appear (`mlflow`, `pickle`, current pointer, production truth)
- permission counters exceed zero

## 6. Integration Design

Existing CR030 APIs remain backward-compatible. New CR139 objects live in the same module because they extend manifest lineage semantics and are consumed by strategy admission packaging later. Existing manifest validation is not changed except for shared helper reuse.

## 7. Test Plan

Add `tests/test_cr139_experiment_manifest_closure.py`:

- Complete closure passes and serializes deterministic fields.
- Missing snapshot/as_of/split/model hash fails closed.
- Mismatched model dataset snapshot hash fails closed.
- Forbidden source markers fail closed.
- Permission counters remain zero and no catalog/lake write is attempted.
- S29 correctness standard ref is required.

Run targeted regression:

- `tests/test_cr139_experiment_manifest_closure.py`
- `tests/test_cr030_experiment_manifest_catalog.py`

## 8. File Ownership

Owned by S12:

- `engine/research_manifest.py`
- `tests/test_cr139_experiment_manifest_closure.py`

No implementation file overlap with S10.

## 9. No-Real-Operation Boundary

S12 writes only process/test artifacts and code. It must not write lake/catalog/manifest pointer/NAS/provider/runtime/Git remote. Runtime tests use in-memory objects only.

## 10. Rollback

Rollback removes the added CR139 dataclasses/helpers/tests. CR030 manifest behavior remains unchanged.

## 11. Acceptance Mapping

- E1: manifest closure references published dataset snapshot, as_of, split, and lineage.
- E2: model artifact hash references the dataset snapshot hash.
- E5: split cutoff, embargo, and label horizon are represented in one policy object and validated together.
- S29 overlap: closure validation explicitly requires S29 correctness standard ref.

## 12. Dependencies

- Requires S11 feature/label artifact boundary for feature artifact refs.
- References S29 correctness standard; implementation does not modify S29 tests.

## 13. Risks

- Existing CR030 manifest tests must remain unchanged and green.
- Avoid overloading catalog publishing behavior; S12 is a closure contract, not a catalog writer.

## 14. CP5 Decision

Proceed to CP6 implementation with fixture/static verification. No additional human gate is required because no real runtime or data-write authorization is introduced.
