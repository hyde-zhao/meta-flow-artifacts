---
story_id: "CR163-S03-two-producer-chain-instrumentation"
stage: "CP6"
status: "implemented"
implemented_by: "meta-dev-debugger"
implemented_at: "2026-07-11T16:30:00+08:00"
design_evidence_ref: "process/stories/STORY-CR163-S03-two-producer-chain-instrumentation-LLD.md"
---

# CR163-S03 Implementation

## Outcome

Implemented the CP5-confirmed S03 v1.1 adapter contract for exactly two producer chains and four frozen mappings. Both chains consume the S01/S02 core unchanged, use one orchestration-owned lineage session and a post-hook orchestration selection writer, and leave both candidate hooks pure. All execution evidence is repository-local and synthetic.

## Preconditions and boundary

- Story state was `dev-ready`; CP5 design evidence was confirmed and S01/S02 dependencies were PASS/PASS_WITH_RISK.
- Process route health was supplied by the Host-created packet/handoff; this worker did not create, repair, or route `process`.
- Writes are limited to the packet's four producer/wrapper files, one synthetic test, and three CP6 evidence outputs.
- No real research run, data/lake/NAS/provider/credential/runtime/statistical/backfill/release/publish/Git-remote operation was executed.
- Platform adaptation is N/A: this is a repository-local Python producer adapter and CLI contract.

## Implemented objects and design-contract mapping

| Object | Frozen contract | Implementation / verification |
|---|---|---|
| `ProducerLineageConfig` | sole keyword-only programmatic DTO; `None` means typed unavailable; dict/string/non-exact types blocked | exact runtime type checks and chain/spec identity binding |
| mapping inventory | CPI-CR163-001..004 exactly 4/4; chain keys exactly `public_stage3` and `legacy_cr039` 2/2 | immutable source inventory plus exact-set test |
| producer session helper | one orchestration owner; declaration before trial; stable parameters+seed identity; one attempt/trial terminal; one selection; request seal in `finally` | synthetic local recorder replay and raw-count assertions |
| public Stage3 chain | CPI-001 wrapper/orchestration + CPI-003 pure hook boundary | Stage3 orchestration opens before `build_strategy_candidate`, writes selection only after return |
| legacy CR039 chain | CPI-002 wrapper/orchestration + CPI-004 pure hook boundary | CR039 orchestration declares aggregate input membership before `build_strategy_candidates`; candidate list length never creates trials |
| canonical CLI parser | one shared `--lineage-spec`/`--lineage-root` pair parser; `00→None`, partial/invalid blocked, valid pair strict-decodes S01 spec | legacy wrapper imports the public parser object; truth/path/schema/identity tests |
| no-inference guardrail | no environment/default/cwd/history/manifest/manual-count inference | parser source inspection and explicit-only path behavior |
| hooks | remain pure; no session/submit/selection responsibility | source inspection for both hooks |

## Minimal implementation slices

1. Added the typed adapter DTO, exact mapping inventory and orchestration-owned local session lifecycle; compiled and replayed a synthetic sealed family.
2. Instrumented public Stage3 around its existing pure candidate hook; no hook signature or core contract changed.
3. Instrumented legacy CR039 around its existing pure list hook; list length is selection output, not family membership or raw count.
4. Replaced the public pass-through CLI with an explicit compatible wrapper owning the canonical strict parser; legacy imports that same parser and only forwards the typed config.
5. Added a synthetic guardrail suite covering exact mapping/chain sets, parser identity/truth table, invalid paths/schema/identity, single session/writer, stable seed identity, exact programmatic type and forbidden inference.

## Verification and handoff

- Packet pytest command: PASS (`210 passed`: 14 S03 adapter tests + 196 unchanged S01/S02 tests).
- Four production files `py_compile`: PASS.
- `git diff --check`: PASS.
- Story return-check and evidence-check: PASS.
- All S03 fixture writes use pytest temporary local roots; no target-project real lineage root was used.
- QA focus: independently exercise full public/legacy fixture entrypoints, injected hook failure retention, same-family replay/conflict, and CLI error precedence.
- Remaining risk: current producer functions perform one hook invocation per orchestration, so the adapter emits ordinal 1; the S01 stable-ID contract proves ordinal exclusion, but a future producer retry loop must explicitly allocate subsequent ordinals under the same trial rather than constructing another adapter session.
- No design delta is required; implementation follows LLD v1.1 and changes no S01/S02 core, HLD, ADR or Feature design.
