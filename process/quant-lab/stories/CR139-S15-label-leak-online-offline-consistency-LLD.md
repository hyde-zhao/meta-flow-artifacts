# CR139-S15 LLD - Label Leak And Offline/Online Consistency

## 1. Story Scope

S15 adds a fixture/static contract for label leakage gates and offline/online feature consistency. It consumes S11 feature/label artifact metadata and S12 manifest closure split/cutoff policy. It does not train models, read real lake data, or start runtime.

## 2. Inputs

- `engine/factor_model_validation.py`
- `engine/factor_robustness.py`
- S11 feature/label artifact contract
- S12 `ExperimentManifestClosure` split policy

## 3. Output Contract

Add validation helpers:

- `label_cutoff_gate(features, labels, *, cutoff_time, label_horizon_days, split_policy_ref)`
- `offline_online_feature_consistency(offline_features, online_features, *, feature_columns=None)`
- `build_offline_online_consistency_gate(...)` wrapper in robustness module

The gate returns a serializable dict with `status`, row counts, violation counts, schema columns, and zero operation counters.

## 4. Validation Rules

- Label leakage blocks when `label_available_at <= feature available_at`.
- Cutoff gate blocks when feature or label availability exceeds the declared cutoff.
- Missing required availability columns blocks.
- Offline/online consistency blocks when feature schema differs or feature values differ for the same primary keys.
- Only deterministic fixture frames are used.

## 5. File Ownership

- `engine/factor_model_validation.py`: label leak/cutoff gate and offline/online consistency core helpers.
- `engine/factor_robustness.py`: lightweight robustness wrapper.
- `tests/test_cr139_label_leak_online_offline_consistency.py`: S15 tests.

## 6. Test Plan

- Passing label/cutoff fixture returns `pass`.
- Future label availability returns `blocked`.
- Future feature/label beyond cutoff returns `blocked`.
- Offline/online schema mismatch returns `blocked`.
- Offline/online value mismatch returns `blocked`.
- Operation counters remain zero.

## 7. Dependencies

- S11 provides label artifact fields.
- S12 provides split/cutoff reference.

## 8. No-Real-Operation Boundary

No provider, lake/catalog/manifest/pointer, NAS, runtime, QMT, simulation/live, credential, dependency, or Git remote operation.

## 9. Acceptance Mapping

- REQ-223/E3: label leakage gate unified with data release/cutoff gate.
- REQ-224/E4: offline/online feature schema and calculation consistency blocks mismatches.

## 10. CP5 Decision

Proceed to CP6 fixture/static implementation. No human gate required.
