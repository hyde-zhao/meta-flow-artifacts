---
status: final
version: "1.0"
scope: "CR163-S03-two-producer-chain-instrumentation"
created_at: "2026-07-11T16:50:00+08:00"
validation_mode: mixed
verification_result: PASS_WITH_RISK
---

# Test Report: CR163-S03

## Summary

| Metric | Result |
|---|---|
| Frozen mappings | `4/4` exact: CPI-CR163-001..004 |
| Deduplicated chains | `2/2` exact: `public_stage3`, `legacy_cr039` |
| Session/selection ownership | one orchestration owner and one post-hook writer per chain; hooks/wrappers submit zero commands |
| Ordering | declaration/helper construction before hook and selection after hook in `2/2` chains |
| Main suite | 210 passed |
| Existing research regression | 12 passed |
| Focused critical cases | 7 passed |
| CLI independent matrix | absent + 10 invalid outcomes + valid typed config all matched |
| Static/compile | PASS |
| Decision | `PASS_WITH_RISK` |

## Scope and objects

The report covers the four S03 production files, the S03 adapter fixture, shared S01/S02 contract/store regression, and the CP6 implementation evidence. It excludes all real data, credentials, runtime/trading, provider, NAS, external framework, backfill, statistical effective-trial computation, publish/release and Git remote operations.

| Object group | Test type | Blocking | Result |
|---|---|---|---|
| two wrappers / shared parser | contract + negative fixture + static | yes | PASS |
| two orchestrations / two hooks | source-order + ownership + regression | yes | PASS |
| `_ProducerLineageTrial` and core session/store | local runtime fixture + replay/contract | yes | PASS |
| CP6 return/evidence | trace/manual | yes | PASS |

## Coverage matrix

| Scenario | Coverage | Evidence | Gap |
|---|---|---|---|
| T-S03-01 public mapping | covered | exact inventory, source order, S03 helper fixture, Stage3 regression | no full instrumented public run fixture |
| T-S03-02 legacy mapping | covered | exact inventory, source order, S03 helper fixture, CR039 regression | no full instrumented legacy run fixture |
| T-S03-03 post-hoc | covered | focused sealed-family late declaration test | none |
| T-S03-04 retry/seed/raw | covered | three attempts raw=1; second seed makes second trial | future producer retry loop conditional gap |
| T-S03-05 no double count/replay | covered | 4 declared members, synthetic hook length 17, replay leaves commands/raw unchanged | none |
| T-S03-06 unavailable | covered | CLI 00→None; exact optional signature/type | none |
| T-S03-07 retention/fail closed | covered | terminal/orphan/incomplete/forbidden core fixtures | none |
| T-S03-08 inventory | covered | exact-set probe | none |
| T-S03-09 permission | covered | local-only command inventory | real runtime intentionally N/A |
| T-S03-10 shared CLI | covered | parser object identity + truth table | none |
| T-S03-11 invalid matrix | covered | partial, URI, missing, bad JSON, non-object, schema, field, mismatch, file root | none |
| T-S03-12 typed/no inference | covered | keyword-only exact signatures, runtime wrong-type rejection, prohibited-source probe | none |

## Commands and results

| ID | Result | Output summary |
|---|---|---|
| full S03/S01/S02 pytest command | PASS | `210 passed in 29.50s` |
| existing Stage3/CR039 regression command | PASS | `12 passed in 0.49s` |
| focused critical pytest selection | PASS | `7 passed in 0.40s` |
| CLI temp-root matrix probe | PASS | exact expected machine outcome for every case; valid pair returned `ProducerLineageConfig` |
| inventory/ownership/order/no-inference probe | PASS | `{mappings:4, chains:2, pure_hooks:2, declaration_before_hook_chains:2, post_hook_writer_chains:2}` |
| `py_compile` + `git diff --check` | PASS | no output/error |

## Design-contract results

| Contract | Result |
|---|---|
| exactly two chains/four mappings and no third chain | PASS |
| one orchestration session owner / selection writer per chain | PASS |
| wrappers only parse/forward; hooks remain pure | PASS |
| declaration before hook/trial work; post-hook selection | PASS |
| attempt ordinal excluded from identity; seed included | PASS |
| identical replay idempotent; conflicting event blocked | PASS |
| post-hoc/orphan/incomplete blocked | PASS |
| explicit CLI pair only; invalid never downgrades to `None` | PASS |
| env/default/cwd/spec-parent/history/manifest/manual count not used to infer lineage | PASS |
| forbidden operations | PASS, all executed counts zero |

## Failure, gaps and residual risk

No failing test and no blocking defect was found. `R-S03-01` is LOW: the current producer paths have no retry loop, so ordinal 1 is the only adapter-emitted attempt. Shared core tests prove three attempts retain raw count 1, but a future producer retry implementation needs a new end-to-end adapter fixture. This is a controlled revisit condition, not a defect in the current one-invocation producer behavior.

## Decision

`PASS_WITH_RISK` → Story may route to `verified-with-risk`; carry `R-S03-01` into CP8/future retry-loop change controls.
