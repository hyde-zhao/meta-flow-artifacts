---
change_id: "CR-158"
release_decision: "READY_WITH_RISK"
status: "draft-pending-cp8-approval"
created_at: "2026-07-05T18:55:00+08:00"
---

# Release Notes - CR158 Event + ML Strategy Adapter

## Summary

CR158 delivers a local/static/fixture adapter slice for event-driven and ML strategy families. It adds a shared adapter core, event-specific extension, ML-specific extension, refs-only evidence handoff helpers, and no-runtime guard counters.

## Delivered

- `engine/strategy_type_adapters.py` with shared core validation, typed event/ML extensions, typed evidence refs, and forbidden operation counter checks.
- Five focused CR158 test modules covering shared core, event extension, ML extension, evidence refs, and no-runtime guard behavior.
- CP6 implementation evidence and CP7 verification evidence for the full S01..S06 slice.

## Quality

- Targeted CR158 tests: `20 passed`.
- Related strategy contract regression subset: `60 passed`.
- `py_compile` for `engine/strategy_type_adapters.py`: PASS.
- `git diff --check` for CR158 implementation/test/process evidence: PASS.

## Release Boundary

Recommended CP8 decision is `READY_WITH_RISK`. This is not a production release, runtime enablement, model registry publication, data lake operation, provider/NAS operation, credential read, QMT/gateway run, simulation/paper/live/trading operation, broker action, external framework run, or Git remote write.
