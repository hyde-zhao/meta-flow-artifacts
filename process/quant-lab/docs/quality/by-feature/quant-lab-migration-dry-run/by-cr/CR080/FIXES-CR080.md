---
status: "none-open"
version: "1.0"
cr_id: "CR-080"
created_at: "2026-06-17T07:14:53+08:00"
---

# FIXES CR080

## Fix Summary

| Fix ID | Source | Severity | Status | Summary |
|---|---|---|---|---|
| FIX-CR080-001 | CP6 first review | INFO | done | `STATE.md` / `CR-INDEX.yaml` were updated from preflight pending to copy executed; `dev-zhu` rerun PASS. |
| FIX-CR080-002 | CP6 first review | INFO | done | CR080 target-root description was updated to state that `reports/` / `data/` are now restored. |

## Open Fixes

| ID | Status | Reason |
|---|---|---|
| N/A | closed | No blocker or required rework remains. |

## Non-Fix Risk Items

| Risk ID | Handling |
|---|---|
| R-CR080-001 metadata-only verification | CP8 risk acceptance, not a fix in this CR. |
| R-CR080-002 NAS/provider/lake/catalog not verified | Follow-up only if user explicitly starts a candidate. |
| R-CR080-003 old root retained | Separate CR required. |
