# Deploy Checklist - CR154

## Release Profile

- Profile: `compact`
- Actual deploy/publish/runtime execution: N/A, not authorized.
- Scope: source and process artifact readiness only.

## Required Before Any Actual Commit / Push / Publish

| Check | Status | Evidence / Note |
|---|---|---|
| Include new source file `engine/cross_strategy_reliability_gates.py` | REQUIRED | Current CP8 treats this as release hygiene risk. |
| Include new test file `tests/research/test_cross_strategy_reliability_gates.py` | REQUIRED | Current CP8 treats this as release hygiene risk. |
| Include modified `engine/strategy_admission_package.py` | REQUIRED | Part of CR154 behavior. |
| Include modified `tests/research/test_strategy_admission_package.py` | REQUIRED | Part of CR154 test coverage. |
| Preserve process artifacts in artifact repo | REQUIRED | `process` is symlink routed. |
| Run target tests | PASS | `process/evidence/CR154-CP7-VERIFICATION.index.json#commands`. |
| Run `git diff --check` | PASS | CP7 evidence. |

## Explicit N/A

- No service deployment.
- No package publish.
- No runtime launch.
- No data migration.
- No catalog/store/registry write.
- No broker/feed/order/reconciliation operation.
- No Git remote write authorized by CP8.
