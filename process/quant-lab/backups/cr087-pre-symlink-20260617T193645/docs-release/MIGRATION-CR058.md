---
status: "draft-for-cp8-review"
version: "1.0"
change_id: "CR-058"
migration_execution_status: "not-authorized"
created_at: "2026-06-14T16:31:54+08:00"
---

# Migration: CR058

## Decision

No real migration is executed in CR058. This document records the migration-readiness boundary for a future execution gate.

## Current Static Gate Assets

| Asset | Path | Status |
|---|---|---|
| Candidate list | `docs/release/CR058-CANDIDATE-LIST.md` | ready for review; all rows `execute_allowed=false`. |
| Preserve-audit allowlist | `docs/release/CR058-PRESERVE-AUDIT-ALLOWLIST.md` | ready; historical evidence protected. |
| Rollback gate | `docs/release/CR058-ROLLBACK-GATE.md` | ready; real execution blocked. |
| No-op guardrails | `docs/release/CR058-NO-OP-GUARDRAILS.md` | ready; 12 not-authorized classes. |

## Future Real Migration Preconditions

| Precondition | Current Status | Required Before Execution |
|---|---|---|
| Frozen candidate manifest | missing | Required. |
| Candidate manifest hash | missing | Required. |
| `pre_cr058_commit` | missing | Required. |
| `pre_cr058_git_bundle` | missing | Required unless separately risk-accepted. |
| `pre_reference_rewrite_manifest` | missing | Required for rewrite. |
| Dirty worktree include/exclude manifest | missing | Required if worktree is dirty. |
| User execution authorization | missing | Required. |

## Preserve-Audit Rule

Historical process evidence remains preserved by default. Any exception must list path, owner, reason, risk, rollback_ref, and approval decision ID.

## Out of Scope

NAS migration, data lake root replacement, trading runtime, provider fetch, lake write, catalog publish, git remote operations, and CR046 resume are out of scope and not authorized.
