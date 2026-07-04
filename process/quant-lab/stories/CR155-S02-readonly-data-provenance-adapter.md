---
story_id: "CR155-S02-readonly-data-provenance-adapter"
cr_id: "CR-155"
title: "Readonly data provenance adapter and guardrails"
status: "lld-ready-for-review"
priority: "P0"
lld_policy: "full-lld"
feature_design_refs:
  - "docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md"
  - "docs/features/runtime-authorization-safety/DESIGN.md"
depends_on:
  - "CR155-S01-baseline-artifact-contract"
lld_ref: "process/stories/CR155-S02-readonly-data-provenance-adapter-LLD.md"
---

# CR155-S02 Readonly Data Provenance Adapter

Define the CR155-scoped readonly provenance contract for local governed lake/current truth input, including proof refs, read scope, forbidden operation counters and downgrade behavior when provenance cannot be proven.

CP5 must review `process/stories/CR155-S02-readonly-data-provenance-adapter-LLD.md` and prove the design cannot be read as lake write, catalog mutation, credential/env/NAS/provider/runtime or trading authorization.
