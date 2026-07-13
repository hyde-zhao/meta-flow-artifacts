---
change_id: "CR-158"
status: "draft-pending-cp8-approval"
created_at: "2026-07-05T18:55:00+08:00"
---

# Feedback - CR158 Event + ML Strategy Adapter

## Feedback Entry Points

| Topic | Route |
|---|---|
| Adapter core too broad or too narrow | Open follow-up CR against FEAT-03 / FEAT-07 adapter contract. |
| Event adapter needs real feed validation | Open a separate runtime authorization CR; do not extend CR158 CP8 approval. |
| ML adapter needs real training or model registry integration | Open a separate runtime/model-registry authorization CR. |
| Evidence refs need additional metadata | Open a follow-up CR preserving refs-only/no-body-copy boundary. |
| Release wording overclaims readiness | Treat as blocker for CP8 approval or create corrective CR before closure. |

## Current Deferred Items

No automatic follow-up is started by CP8 approval. Future real feed/training/runtime/registry/publish/trading validation requires a separate explicit gate.
