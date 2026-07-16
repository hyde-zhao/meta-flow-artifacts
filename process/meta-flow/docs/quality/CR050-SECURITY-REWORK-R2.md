---
cr_id: "CR-050"
status: "resolved"
review_stage: "CP7-convergence"
resolved_at: "2026-07-16"
---

# CR-050 Security Rework R2

## Findings and resolution

| Finding | Severity | Resolution | Verification |
|---|---|---|---|
| Git `check-ref-format` accepts shell-metacharacter names such as `foo;bar` and `foo$(id)` | HIGH | Added an allowlisted raw-input grammar before Git validation; unsafe branch/remote input fails with `invalid_branch/invalid_input` before lifecycle mutation. | Five unsafe input forms rejected in unit fixture. |
| Open plan→execute gap could observe a newer remote default than the OID bound into authorization | BLOCKER | Fetch all, re-query every remote default against the planned OID, then pull and create from the exact full OID; any drift is `BLOCKED` and creates zero CR refs. | Concurrent-default fixture advances remote after fetch; 2/2 local/remote CR ref count remains zero. |
| Finish plan→execute gap did not re-prove remote CR identity/default ancestry immediately before recovery/delete | BLOCKER | Added all-repository reproof before recovery refs and per-repository reproof before each delete. CR drift blocks before recovery or delete. | Post-plan CR-tip drift fixture preserves 2/2 branches and creates zero recovery refs. |

## Regression result

`tests/test_git_branch_lifecycle.py`, `tests/test_workspace_git_sync.py` and `tests/test_cr_lifecycle.py`: 49 passed. Touched-path Ruff and `git diff --check`: PASS. No current project/artifact remote operation was executed.

The correction changes implementation detail only; HLD/LLD invariants and artifact-first decision remain unchanged. Prior CP6/CP7 results are preserved and superseded by R2 results with current hashes.
