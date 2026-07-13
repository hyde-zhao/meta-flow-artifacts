# CR-164 Deploy Checklist

## Pre-deploy

- [x] Five CP6 results validate and are ledgered.
- [x] Five CP7 results validate and are ledgered.
- [x] Affected tests 109/109 PASS; new provenance 7/7.
- [x] No dependency, installer, environment variable, database or migration change.
- [x] Runtime/data/external operation counters remain 0.
- [x] CP8 user accepts `READY_WITH_RISK` and the two named risks.
- [ ] If repository delivery is desired, obtain separate explicit authorization for paired source + artifact commit/push.

## Post-deploy

- Confirm no consumer treats `typed_unavailable` as PASS.
- Confirm evidence provenance includes candidate membership hash, lineage ref/hash and all method parameters.
- Confirm no effective-trial field is populated by raw-count DSR.
- Re-run the 109-test affected subset after any merge conflict resolution.

No true deployment action is part of this checklist execution.
