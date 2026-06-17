# CR068 Repository Hygiene Forward-fix Plan

## Purpose

Close the CR067 residual risk that the public `quant-lab` baseline does not include `.gitignore`.

## Scope

In scope:

- Treat the current tracked `.gitignore` as the candidate hygiene file.
- Add `.gitignore` to the public baseline only after CR068 CP2 / CP3 / CP5 are approved.
- Run a fail-closed hygiene scan before any commit / push.
- If approved, perform an exact remote ordinary fast-forward forward-fix against `git@github.com:hyde-zhao/quant-lab.git` `master`.
- Record CP6 / CP7 / CP8 evidence after execution.

Out of scope:

- Branch rename or default branch governance.
- Force push, tag, history rewrite, remote deletion, remote set-url.
- NAS/data lake/provider/catalog/runtime/credential operation.
- `.gitignore` broad rewrite beyond the reviewed tracked file.
- CR046 recovery.

## Approved-Only Execution Sequence

This sequence is not authorized until CR068 CP2 / CP3 / CP5 are all `approved`.

1. Re-read CR068 CP2 / CP3 / CP5 checkpoints and confirm approved.
2. Re-read CR067 release context and expected remote baseline commit.
3. Create a temporary working directory outside the current worktree.
4. Clone or fetch only the exact remote `git@github.com:hyde-zhao/quant-lab.git` `master`.
5. Verify the remote baseline starts from `495fcc495170454d709626dcde0923d8fa38c57a` or record a fail-closed mismatch.
6. Copy the current tracked `.gitignore` into the temporary checkout.
7. Run the CR068 scan gate.
8. Commit only `.gitignore` if and only if the scan passes.
9. Push only a normal fast-forward update to `master`.
10. Record remote verification and route to CP6 / CP7 / CP8 evidence.

## Stop Conditions

- Any CP2 / CP3 / CP5 checkpoint is not `approved`.
- Remote branch does not match the expected baseline and no explicit DQ authorizes handling it.
- `.gitignore` scan finds credential exposure, dangerous unignore rules, oversized file inclusion, or ignored-public-evidence conflict.
- Push would require force or non-fast-forward.
- The operation would require branch/default branch governance.

## Verification

- Manifest-vs-snapshot check: only `.gitignore` is added to the public baseline.
- Scan gate: PASS with 0 blocking findings.
- Remote verification: `refs/heads/master` points to the new forward-fix commit.
- No force/tag/rewrite/branch/default branch governance evidence.

## Rollback / Forward-fix

No force push or history rewrite is allowed. If the forward-fix is wrong after publication, open a new remediation CR and publish a new ordinary forward-fix commit.
