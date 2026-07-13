# CR155 Rollback

Rollback is source-level only.

| Trigger | Action | Verification |
|---|---|---|
| Artifact contract semantics found incorrect | Revert `engine/daily_multifactor_baseline_artifact.py` and `tests/research/test_daily_multifactor_baseline_artifact.py`; reopen CR155 or create follow-up CR. | Rerun related regression suite. |
| Overclaim wording escapes tests | Mark CR155 `NOT_READY`, add regression test for wording, rerun CP6/CP7. | CR155 targeted tests pass. |
| User rejects CP8 risk acceptance | Keep CR155 active and create `FU-CR155-001` candidate for real-data validation evidence. | CP8 checkpoint records changes requested. |

No data migration rollback is needed because no lake/catalog/store/registry write was executed.
