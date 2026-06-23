---
title: "CR135 Release Notes"
source_cr: "CR-135"
status: "ready_with_risk"
profile: "compact"
created_at: "2026-06-23"
---

# CR135 Release Notes

## Summary

CR135 adds an offline Strategy Runner artifact bundle workflow. Successful offline runs can write a local bundle with `run-result.json`, `runner-evidence.index.json`, `run-spec.snapshot.json` and `manifest.json`.

## User-Visible Change

- `RunSpec` accepts optional `bundle_output_path`.
- `run_strategy_package_from_path(..., bundle_output_path=...)` writes a pass-only local bundle.
- CLI supports `--bundle-output <dir>` and `--inspect-bundle <dir>`.
- Bundle replay / inspect reads local bundle files and does not rerun the strategy package.

## Compatibility

Existing CR128 / CR133 / CR134 runner flows remain compatible. Bundle output is optional.

## Known Risk

`meta-flow check cr-tracking` exits OK and reports `active formal CRs: CR-135`, but still prints a non-blocking warning that `CR-INDEX.yaml` does not mention active formal CR CR135.

## Not Authorized

CP8 approval does not authorize runtime, NAS, QMT, credentials, provider/lake/catalog, trading actions, commit, push or remote write.

## Next Recommended Action

If CP8 is approved, close CR135 as `READY_WITH_RISK`. If zero-warning tracking is required, start a separate cleanup follow-up.
