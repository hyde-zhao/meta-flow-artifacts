# CR068 Rollback / Forward-fix

## Policy

CR068 does not authorize force push, history rewrite, remote deletion or remote set-url. If the published `.gitignore` needs correction, use a new ordinary forward-fix commit under a new or amended CR.

## Current Published Commit

| 字段 | 值 |
|---|---|
| Remote | `git@github.com:hyde-zhao/quant-lab.git` |
| Branch | `master` |
| Commit | `3ade165e8b1edad031a911490cf6c1cee942616e` |

## Allowed Recovery Route

1. Open a remediation CR.
2. Freeze the corrected `.gitignore` or repo hygiene delta.
3. Run scan gate.
4. Push a normal fast-forward commit.

## Forbidden Recovery Route

- No force push.
- No tag rewrite.
- No history rewrite.
- No remote deletion.
- No remote set-url.
