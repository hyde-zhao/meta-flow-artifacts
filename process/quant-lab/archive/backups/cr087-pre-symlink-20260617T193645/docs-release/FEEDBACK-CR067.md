# CR067 Feedback

## Observation Signals

| Signal | Threshold | Action |
|---|---|---|
| Remote branch/default branch mismatch | User wants `main` or GitHub default branch setup | Start branch governance CR; do not rename in CR067. |
| Missing repo hygiene files | User wants `.gitignore`, license, CI, or process evidence added | Start clean-publication follow-up CR. |
| Public snapshot content issue | Any sensitive or unwanted file found | Stop further changes; open remediation CR; no force push without explicit authorization. |
| Package/import issue | Import smoke fails in consumer environment | Open package/import follow-up CR. |

## Feedback Routing

- Security or sensitive file issue: immediate blocker, new remediation CR.
- Branch/default branch issue: branch governance CR.
- Missing docs or repo hygiene: low-risk follow-up CR.
- Runtime/data/NAS issue: CR063/CR064/CR065 family, not CR067.
