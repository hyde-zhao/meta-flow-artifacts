---
handoff_id: "CR163-CP5-S03-LLD-REWORK-RETURN-20260711"
workflow_id: "CR-163"
change_id: "CR-163"
story_id: "CR163-S03-two-producer-chain-instrumentation"
stage: "story-planning"
checkpoint: "CP5"
canonical_role: "meta-dev"
status: "PASS"
completed_at: "2026-07-11"
implementation_performed: false
clarification_count: 0
---

# CR163 CP5 S03 LLD Rework Return Summary

## Outcome

PASS. Reworked only `process/stories/STORY-CR163-S03-two-producer-chain-instrumentation-LLD.md` and stopped before implementation or CP5 approval.

## Frozen public contract

- Programmatic producer entrypoints accept only keyword-only `lineage_config: ProducerLineageConfig | None = None`.
- CLI accepts only the pair `--lineage-spec <local-json-path>` and `--lineage-root <explicit-local-output-root>`.
- Both CLI arguments absent maps to `None`, `typed_unavailable`, and zero session/present outputs.
- Both arguments present invoke one shared parser before the first trial and strictly decode `ExperimentFamilySpec`.
- Partial pairs, invalid/non-local paths, invalid JSON, unsupported schema, missing/invalid required fields, and chain identity mismatch are blocked and can never produce `present`.
- Invalid input cannot silently degrade to `None`; bare mapping/string programmatic values are blocked.
- Environment variables, default directories, cwd/spec-parent discovery, historical artifacts/manifests, and manual count are prohibited inference sources.
- Public Stage3 and legacy CR039 wrappers reuse one parser and one machine-error contract.

## Updated evidence

The S03 LLD revision is now `1.1`. Requirements, module ownership, data/config model, interfaces, flow, technical rules, failure table, security controls, tests T-S03-10..12, TASK mappings, risks, rollback triggers, and DoD all reflect the hardened public contract.

## Validation

| Check | Result |
|---|---|
| `uv run meta-flow story lld-check --lld process/stories/STORY-CR163-S03-two-producer-chain-instrumentation-LLD.md --evidence-type full-lld` | PASS — `LLD Structure Check: OK` |
| `git diff --check` scoped to the S03 LLD during drafting | PASS |

## Routing

- No clarification or blocker was found.
- Exact next route: Host Orchestrator reruns the S03 CP5 implementability precheck, refreshes the unified CP5 evidence/decision brief, and keeps implementation blocked until CP5 approval and S01/S02 dev gates are satisfied.

