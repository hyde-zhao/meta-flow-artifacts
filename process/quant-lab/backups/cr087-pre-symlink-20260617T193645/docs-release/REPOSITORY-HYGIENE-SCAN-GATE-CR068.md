# CR068 Repository Hygiene Scan Gate

## Objective

Prevent a repo hygiene forward-fix from weakening public safety controls while adding `.gitignore` to the published baseline.

## Required Checks

| Check | PASS Condition | Fail-closed Condition |
|---|---|---|
| `.gitignore` source | Candidate is the current tracked `.gitignore` | Candidate missing or untracked without explicit DQ |
| Secret protection | `.env`, `.env.*`, credentials, keys remain ignored except `.env.example` | Any rule would publish local credentials |
| Data protection | `data/`, `reports/`, `runs/`, parquet/db/cache patterns remain protected | Rule would unignore real data or reports |
| Evidence exceptions | Only intended public-safe exceptions exist | Broad `!` pattern exposes hidden or large files |
| Delta scope | Only `.gitignore` changes in forward-fix | Additional files appear unexpectedly |
| Remote baseline | Remote `master` starts from expected CR067 commit | Mismatch requiring force, rewrite, branch governance, or manual conflict resolution |

## Stop Policy

Any BLOCKING or HIGH finding stops execution before commit / push. False positives require revising CR068 Decision Brief or opening a remediation CR.

## Non-authorized

- No branch/default branch governance.
- No force push, tag, history rewrite, remote deletion or remote set-url.
- No credential read.
- No NAS/data lake/provider/catalog/runtime operation.
- No physical move, bulk import rewrite or CR046 recovery.
