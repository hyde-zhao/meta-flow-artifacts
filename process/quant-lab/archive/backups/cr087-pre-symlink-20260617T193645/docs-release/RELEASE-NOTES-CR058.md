---
status: "draft-for-cp8-review"
version: "1.0"
change_id: "CR-058"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "full"
created_at: "2026-06-14T16:31:54+08:00"
---

# Release Notes: CR058 Repo-local Mechanical Migration / Relayout Gate

## Summary

CR058 delivers a static-only / no-op gate package for future repo-local mechanical migration and relayout. It does not execute migration.

## Included

| Artifact | Path | Purpose |
|---|---|---|
| Candidate list | `docs/release/CR058-CANDIDATE-LIST.md` | Candidate schema, owner classification, fail-closed rules. |
| Preserve-audit allowlist | `docs/release/CR058-PRESERVE-AUDIT-ALLOWLIST.md` | Historical evidence protection and sensitive filter. |
| Rollback gate | `docs/release/CR058-ROLLBACK-GATE.md` | Required rollback refs and execution preflight blockers. |
| No-op guardrails | `docs/release/CR058-NO-OP-GUARDRAILS.md` | 12 not-authorized operation classes. |
| Implementation evidence | `process/stories/CR058-BATCH-A-IMPLEMENTATION.md` | CP6 static implementation trace. |
| Verification evidence | `docs/quality/VERIFICATION-REPORT-CR058.md` | CP7 static-only verification. |

## Release Decision

| Field | Value |
|---|---|
| Decision | `READY_WITH_RISK` |
| Scope | Static gate delivery only. |
| Not included | Real file move, rename, delete, rewrite, NAS, lake, git remote, credentials, runtime, provider fetch, CR046 resume. |
| Next gate | CP8 human review. |

## Known Risks

| Risk ID | Summary | CP8 Handling |
|---|---|---|
| R-CR058-01 | rollback commit / bundle / manifest / candidate hash missing. | Accept for static-only; keep real execution blocked. |
| R-CR058-02 | CP7 PASS_WITH_RISK may be misread as execution authorization. | CP8 explicitly lists `NA-CR058-001..012`. |
| R-CR058-03 | Historical `local_backtest` references remain. | Accept under preserve-audit policy. |
| R-CR058-04 | CR046 remains paused active formal CR. | Do not resume unless user explicitly restores CR046. |

## Compatibility

No code, package, import path, data lake root, runtime, environment variable, installer, or platform behavior changes are included.

## User Action

Review `process/checkpoints/CP8-CR058-DELIVERY-READINESS.md`. Reply `approve`, `修改: <具体修改点>`, or `reject`.
