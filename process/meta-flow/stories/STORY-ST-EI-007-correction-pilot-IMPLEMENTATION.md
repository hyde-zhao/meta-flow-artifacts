---
status: "completed"
version: "1.0"
story_id: "ST-EI-007"
implementation_type: "append-only-correction"
source_story: "process/stories/STORY-ST-EI-007-correction-pilot.md"
source_design_evidence: "process/stories/STORY-ST-EI-007-correction-pilot-LLD.md"
created_by: "host-orchestrator-inline-fallback"
---

# Implementation: ST-EI-007 — append-only correction / pilot guard

The v1 correction contract requires a typed target, source hash, constrained annotation/evidence/provenance patch, author, reason and evidence. `append_correction` checks the ledger prefix hash, fsyncs one canonical event, verifies its tail, and returns a receipt; replay produces a derived copy only. The CR-163 adapter only plans and preflights a synthetic, exactly-23-target dry run. Any real apply is blocked without separate external authorization, and no quant-lab target was read or written.
