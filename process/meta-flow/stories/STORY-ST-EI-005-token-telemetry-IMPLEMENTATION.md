---
status: "completed"
version: "1.0"
story_id: "ST-EI-005"
implementation_type: "evidence-contract"
source_story: "process/stories/STORY-ST-EI-005-token-telemetry.md"
source_design_evidence: "process/stories/STORY-ST-EI-005-token-telemetry-LLD.md"
created_by: "host-orchestrator-inline-fallback"
---

# Implementation: ST-EI-005 — truthful token telemetry

`TokenUsage` provides three mutually exclusive measurement states: `measured`, `unavailable`, and `estimated`. Only `measured + platform-reported + total_tokens` contributes to a token total. Optional dispatch `usage` receipts are parsed without turning missing data or character estimates into measurements; machine audit aggregation exposes unavailable state explicitly.

| Contract | Implementation | Verification |
|---|---|---|
| no estimate masquerades as fact | `validate_usage` | measured-source negative fixture |
| per-dispatch receipt surface | `usage_from_event` | absent/available receipt fixture |
| independent aggregation | `aggregate_usage` | state-count fixture |

Current platform telemetry remains unavailable; this Story deliberately records that limitation instead of inferring a total.
