# CR155 Feedback

## Observation Signals

| Signal | Threshold | Route |
|---|---|---|
| Artifact contract ambiguity | Any downstream consumer cannot distinguish artifact, admission package and rerun evidence | Create follow-up CR or reopen CR155 before CP8 approval. |
| Overclaim wording request | Any request to call the artifact paper/live/trading/runtime/production ready | Block and route to runtime authorization / release readiness CR. |
| Strategy admission remediation requested | User wants the baseline to become `paper_candidate=true` | Create `FU-CR155-001` or continue CR155 with strategy remediation and fresh CP7 evidence. |
| Real-data rerun drift | Future rerun differs from `cr155-real-lake-baseline-rerun-a/b` core metrics | Block paper_candidate claims and investigate data/code/config drift. |

## Follow-Up Candidate

| ID | Status | Summary |
|---|---|---|
| FU-CR155-001 | candidate-if-needed | Remediate or redesign the daily multifactor baseline so economic_significance and out_of_sample_validation can pass, then rerun admission and CP7 evidence. |
