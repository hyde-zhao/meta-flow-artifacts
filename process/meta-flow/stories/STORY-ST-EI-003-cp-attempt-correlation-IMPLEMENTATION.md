---
status: "completed"
version: "1.0"
story_id: "ST-EI-003"
implementation_type: "checker-contract"
source_story: "process/stories/STORY-ST-EI-003-cp-attempt-correlation.md"
source_design_evidence: "process/stories/STORY-ST-EI-003-cp-attempt-correlation-LLD.md"
created_by: "host-orchestrator-inline-fallback"
---

# Implementation: ST-EI-003 — CP result / final-attempt correlation

`cp result-check` now supports `audit|strict` correlation profiles. It hashes declared inputs, rejects unsafe/stale hash refs, validates append-only result attempts and supersession references, and refuses to use untyped or nonterminal dispatch evidence as a final successful attempt. Compatibility mode remains read-only for legacy results; audit reports unavailable evidence, while strict fails closed.

## Contract mapping and verification

| Contract | Implementation | Verification |
|---|---|---|
| result attempt/hash/supersedes | `meta_flow/checks/cp_result.py` correlation findings | strict negative fixtures |
| final typed attempt only | typed event-ledger consumption | missing/unfinished attempt fixture |
| phase/gate separation | ST-EI-001 state-transition reuse | 67 tests / 44 subtests |
| no false platform proof | no receipt synthesis | A-baseline retained |

No historical ledger/result was rewritten; no runtime, credential, publish, commit/push, or quant-lab business source was touched.
