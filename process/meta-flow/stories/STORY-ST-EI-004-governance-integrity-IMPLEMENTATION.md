---
status: "completed"
version: "1.0"
story_id: "ST-EI-004"
implementation_type: "governance-guardrail"
source_story: "process/stories/STORY-ST-EI-004-governance-integrity.md"
source_design_evidence: "process/stories/STORY-ST-EI-004-governance-integrity-LLD.md"
created_by: "host-orchestrator-inline-fallback"
---

# Implementation: ST-EI-004 — governance, finalization, compaction

Delivered current state now rejects active Story/context/delegation refs and checks that a declared workflow-health report actually exists and has counters. Read expansions now distinguish outside-default reads from their authorization. Ledger compaction produces typed semantic manifests using `event_id`, `dispatch_id`, `attempt_id`, and `run_id` in their own namespaces, verifies an untouched restore candidate before applying, and never falls back between them.

| Contract | Implementation | Verification |
|---|---|---|
| delivered no active work | current-state invariant | state negative fixture |
| health ref exists/parseable | `check_current_state` | strict state check |
| explicit read authorization | read-expansion fields | context fixture |
| compaction preserves semantic proof | `semantic_manifest` + restore candidate | no-fallback fixture |

Existing historical read rows are reported as legacy (warning), not rewritten.
