# CR-046 CP2 Scope Review R1

- Review date: 2026-07-12
- Reviewer: user
- Host disposition: `changes_requested`
- Scope decision: all three omissions are accepted as required scope, not deferred.

## Accepted findings

| Finding | Severity | Verified fact | Required routing |
|---|---|---|---|
| CR046-SCOPE-F01 Compaction interaction | HIGH | `ledger_compaction.py` conflates `event_id`, `dispatch_id` and `run_id` as fallback display identifiers and has no attempt-chain/workflow-health preservation invariant | Add to Story 2/4 contract and CP7 compaction replay fixtures |
| CR046-SCOPE-F02 Post-close correction lifecycle | HIGH | CR-163 pilot exists, but no reusable correction event schema, allowed post-CP8 mutation boundary or correction audit trail is in scope | Formalize lifecycle in Story 7 and make pilot consume it |
| CR046-SCOPE-F03 Machine-generated audit report | HIGH | Existing audit comparisons were hand-written and contained event/attempt and token-share counting errors | Add a provenance-bearing generated audit artifact to Story 4/6 |
| CR046-SCOPE-F04 CP1/CP2 provenance dogfooding | MEDIUM | CR046 CP1/CP2 results pass with `checker_provenance` absent because the validator treats it as optional | Add current results as first-class Story 6 negative fixtures; migrated/current results must gain measured checker provenance without rewriting original records |
| CR046-SCOPE-F05 Dispatch attestation limitation | MEDIUM | `/root/pm_cr046` and `tool_name=spawn_agent` are session-observed but not backed by a repository-verifiable platform receipt | Disclose in CP2; Story 2 must define evidence levels and unavailable receipt behavior |

## Required acceptance additions

1. Compaction preserves event/dispatch/attempt relationships, terminal attempt selection, correction chains and workflow-health refs; restore/replay proves no semantic loss.
2. Post-close corrections use a versioned append-only schema, explicit allowed fields/scope, author/reason/evidence/supersedes links and an independently checkable audit trail.
3. Meta Flow generates a machine-readable audit report from ledgers/results with event-row, attempt, thread, outcome and measured/proxy/unavailable token metrics kept separate; the report carries checker provenance and input hashes.
4. CR046 CP1/CP2 null-provenance evidence is retained as historical input and must fail the new strict profile or be reported as legacy/unavailable; it must not be silently rewritten.
5. Current Codex dispatch evidence is reported as `session-observed/repository-unverifiable` until a platform receipt is available.
