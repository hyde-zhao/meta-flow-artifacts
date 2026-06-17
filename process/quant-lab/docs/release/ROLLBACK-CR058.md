---
status: "draft-for-cp8-review"
version: "1.0"
change_id: "CR-058"
rollback_scope: "static-documents-only"
created_at: "2026-06-14T16:31:54+08:00"
---

# Rollback: CR058

## Current Rollback Scope

Current CR058 delivery only creates static documents and process evidence. No real migration was executed, so runtime/data rollback is N/A.

## Document-Level Rollback

| Trigger | Rollback Action | Verification |
|---|---|---|
| CP8 rejected | Mark CR058 as not ready or changes requested; keep files as draft evidence. | Re-run CP8 after changes. |
| Release context wrong | Update `process/release/RELEASE-CONTEXT-CR058.yaml`. | YAML parse and human-gate check. |
| Risk classification wrong | Update CP8 checkpoint Decision Brief. | Re-run human-gate validation. |

## Real Execution Rollback Gate

Real execution remains blocked until all required refs exist:

| Required Ref | Current Status | Effect |
|---|---|---|
| `pre_cr058_commit` | missing / planned | Blocks real move / rewrite. |
| `pre_cr058_git_bundle` | missing / planned | Blocks irreversible bulk action unless separately waived. |
| `pre_reference_rewrite_manifest` | missing / planned | Blocks reference rewrite. |
| `candidate_manifest_hash` | missing / planned | Blocks candidate freeze. |

## Not Authorized Rollback Actions

`git reset --hard`, `git checkout --`, history rewrite, remote rename, push, tag, NAS restore, lake restore, and runtime recovery are not authorized by CR058 CP8.
