---
story_id: "STORY-CR049-S01"
cr_id: "CR-049"
title: "Archive and retire merged auxiliary branches"
status: "verified-with-risk"
lld_policy: "waived"
lld_waiver_reason: "Process-lite Git ref hygiene action with exact refs and no source-code or design change."
---

# STORY-CR049-S01 Archive and retire merged auxiliary branches

## Acceptance Criteria

- Two annotated archive tags peel to the exact former branch tips.
- Only the two named remote auxiliary branches are deleted.
- Only `main` remains as a local and remote branch.
- `main` remains at `ad3f6f42b179ec4767d29a07b7e9604e1969b8fe`, and `d266e05660be2d7b14e74b09ee2819402565103e` remains reachable and an ancestor.
- No source worktree change and no `process/quant-lab/**` inclusion occurs.

## Technical Note

The operation is tag-before-delete: create annotated tags, push and verify the remote peeled refs, then delete exact branch refs. Any ref drift or tag conflict blocks deletion. Recovery recreates the former branch from its immutable archive tag.
