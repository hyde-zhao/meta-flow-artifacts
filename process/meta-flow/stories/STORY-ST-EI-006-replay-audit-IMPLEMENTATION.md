---
status: "completed"
version: "1.0"
story_id: "ST-EI-006"
implementation_type: "replay-audit"
source_story: "process/stories/STORY-ST-EI-006-replay-audit.md"
source_design_evidence: "process/stories/STORY-ST-EI-006-replay-audit-LLD.md"
created_by: "host-orchestrator-inline-fallback"
---

# Implementation: ST-EI-006 — checker replay and machine audit

Replay provenance distinguishes an unavailable historical checker from a current-checker decision. The new machine audit has independent event-row, typed-attempt, thread, terminal-event, and token dimensions. Legacy `codex_agent_name` is permanently classified as D3 self-declared/unverifiable. D0 freshness re-probes on expiry, session, epoch, config, selector schema, or reload change.

The implementation is read-only over historical evidence and keeps repository replay and runtime platform attestation as separate axes.
