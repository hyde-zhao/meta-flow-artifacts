---
checkpoint: "CP6"
cr_id: "CR-131"
title: "Design Surface Archive Cleanup Implementation Evidence"
status: "PASS"
created_at: "2026-06-23T18:12:00+08:00"
owner: "host-orchestrator"
context_ref: "process/context/CP6-CR131.context.json"
summary_ref: "process/changes/summaries/CR-131.summary.json"
authorization: "design docs archive, archive index, lightweight checker/tests and process evidence only"
---

# CP6 - CR131 Design Surface Archive Cleanup Implementation Evidence

## Entry Criteria

| Item | Result | Evidence |
|---|---|---|
| CR130 closed | PASS | `process/changes/summaries/CR-130.summary.json` shows closed/ready |
| CR131 active and authorized | PASS | `process/changes/CR-131-DESIGN-SURFACE-ARCHIVE-CLEANUP-2026-06-23.md` |
| Scope excludes external permission domains | PASS | CR131 `## 不授权范围` |
| Machine context exists | PASS | `process/context/CP6-CR131.context.json` |

## Implemented Objects

| Object | Action | Notes |
|---|---|---|
| `process/docs/design/*CR*.md` | moved | 30 CR-named historical design docs moved to `process/archive/design-cr-docs/`. |
| `process/archive/design-cr-docs/ARCHIVE-INDEX.md` | added | Archive index for moved CR-named design docs. |
| `process/docs/design/ARCHIVED-DESIGN-INDEX.md` | added | Default design root entry pointing to archive root and archive index. |
| `process/docs/design/HLD.md` | updated | v1.3 / CR131; references archived CR046 / CR126 paths. |
| `process/docs/design/ARCHITECTURE-DECISION.md` | updated | v1.4 / CR131; references archived CR046 / CR053 / CR126 paths. |
| `process/docs/design/BLUEPRINT.md` | updated | v1.3 / CR131; source HLD for CR051 points to archive. |
| `process/docs/design/FEATURE-DESIGN-MATRIX.md` | updated | v1.4 / CR131; CR046/CR051/CR053 references marked archived where relevant. |
| `process/archive/design-cr-docs/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | updated | Internal CR126 / ADR refs point to archive paths. |
| `process/archive/design-cr-docs/ARCHITECTURE-DECISION-CR046.md` | updated | Internal CR126 / HLD refs point to archive paths. |
| `scripts/check_design_runner_boundary.py` | updated | Reads CR046/CR126 historical docs from archive root. |
| `tests/test_cr130_design_runner_boundary.py` | updated | Covers archive-aware runner boundary check. |
| `scripts/check_design_surface.py` | added | Checks default design root has only allowed current files and no CR-named docs. |
| `tests/test_cr131_design_surface.py` | added | Covers current pass, CR-named root file failure and CLI JSON output. |

## Archive Result

| Surface | Result |
|---|---|
| Default design root | 9 allowed current files: `ARCHITECTURE-DECISION.md`, `ARCHIVED-DESIGN-INDEX.md`, `BLUEPRINT.md`, `DEPENDENCY-MAP.md`, `DOMAIN-MAP.md`, `FEATURE-DESIGN-MATRIX.md`, `HLD.md`, `MODULE-BOUNDARIES.yaml`, `PACKAGE-IDENTITY.yaml` |
| Archive root | 31 files: 30 historical CR design docs plus `ARCHIVE-INDEX.md` |

## Non-Authorized Boundary

CR131 did not authorize or execute:

- `.env`, token, secret, credential, account, funds, positions, orders, fills, session, cookie or raw log reads.
- NAS mount/list/copy/read/write/publish/pull/delete.
- QMT, MiniQMT, XtQuant or gateway runtime startup/connection.
- Provider fetch, lake write, catalog publish or current pointer modification.
- submit/cancel/buy/sell, simulation or live trading.
- `.gitignore` changes, commit, push or Git remote write.

## Exit Criteria

| Item | Result | Evidence |
|---|---|---|
| CR-named design docs removed from default root | PASS | `scripts/check_design_surface.py` |
| Historical docs retained | PASS | `process/archive/design-cr-docs/ARCHIVE-INDEX.md` |
| Runner architecture checker still passes | PASS | `scripts/check_design_runner_boundary.py` |
| Guardrail tests added | PASS | `tests/test_cr131_design_surface.py` |

## Decision

CP6 result: PASS. CR131 is ready for CP7 verification.
