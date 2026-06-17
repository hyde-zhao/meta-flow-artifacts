# CR069 CR062 Supersession Ledger Cleanup Plan

## Scope

CR069 is a ledger-only cleanup CR. Its target is to decide and later record that CR062's original current-branch publication route was replaced by CR066/CR067/CR068.

## Proposed Final State

- CR062 remains historically blocked by tracked forbidden publication paths.
- CR062 gets classified as `superseded-by-clean-publication-baseline` or equivalent closed / replaced ledger state after CP5 approval.
- CR066, CR067 and CR068 stay closed-current-delivery / READY_WITH_RISK.
- CR046 remains user-paused.

## Execution Boundary

No operation in CR069 may run:

- `git remote add`, `git remote set-url`, `git push`, `git tag`, branch rename, default branch governance, history rewrite or force push.
- NAS/data lake/provider/catalog/runtime operations.
- Credential, `.env`, token, SSH agent, account or trading reads.
- Physical move / rename / delete or bulk import rewrite.
- CR046 recovery.

## Verification

After CP5 approval and ledger-only updates:

- `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .`
- `uv run --python 3.11 python scripts/check_human_gate_decision_brief.py ...`
- `git diff --check`

## Rollback

Rollback is ledger-only: revert CR062 classification and CR tracking wording to the current blocked baseline while keeping CR066/CR067/CR068 closed evidence unchanged.
