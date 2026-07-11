---
story_id: "CR163-S02-recorder-seal-supersession"
stage: "CP6"
status: "implemented-rework-round-2"
implemented_by: "meta-dev-debugger"
implemented_at: "2026-07-11T13:43:00+08:00"
design_evidence_ref: "process/stories/STORY-CR163-S02-recorder-seal-supersession-LLD.md"
---

# CR163-S02 Implementation

## Outcome

Implemented the CP5-confirmed S02 v1.1 contract as a repository-local, stdlib-only store. The implementation preserves the verified S01 public contract unchanged and adds only the downstream recorder/sealer/resolver module and its synthetic `tmp_path` tests.

## Preconditions and boundary

- S01 dependency: CP7 PASS; imported as the sole owner of frozen lineage objects, commands, fold, and semantic validation.
- Story gate: `dev-ready`, `design_evidence_confirmed=true`, dependency and file-conflict gates satisfied.
- Writes: exactly the five packet-authorized outputs.
- Runtime boundary: no real lineage root, lake, NAS, provider, credential, runtime, registry, external framework, statistical correction, CR155 backfill, Git remote, publish, or release operation.
- Platform adaptation: N/A; this Story is a local Python module and `tmp_path` fixture suite.

## Implemented objects and contract mapping

| Object | Implemented contract | Verification |
|---|---|---|
| Restricted canonical JSON/JSONL | Recursive key sorting, UTF-8/no whitespace, plain finite decimal normalization, LF-delimited commands, invalid type/key/non-finite rejection | golden and rejection tests |
| `LocalFamilyLineageRecorder` | create-only spec, `O_APPEND` one-write+fsync events, per-family lock, rebuilt event index, identical receipt replay, duplicate-conflict fail closed | open/submit/rebuild tests |
| Immutable publisher | same-filesystem temp+fsync+hard-link no-clobber, `O_EXCL` local fallback, identical replay, conflicting final rejection, no replace | immutable/collision tests |
| Seal | domain-separated SHA-256 envelope excluding path/mtime/clock/`sealed_at`/hash itself; S01 fold+validator; sealed count/sequence prefix boundary | 10-root determinism, target binding, hash-domain tests |
| Supersession resolver | all-manifest discovery; embedded version/ref/hash validation; prefix hash recomputation; validation replay; broken/cycle/fork/tamper detection; unique head | v1→v2, preserved v1 replay, broken/cycle/fork/tamper tests |
| No-resume boundary | malformed/partial ledger is retained and blocks reopen; no truncate/repair/resume API or cursor; recovery requires a distinct family starting at declaration; counts are not merged | 9,999-complete-plus-partial fixture and API inspection |
| CP7 path hardening | Every existing path component, including the family directory itself, is rejected if it is a symlink before any child mkdir/write | hostile family-link and nested-ref fixtures |
| CP7 request binding | Superseding request version/ref/hash/reason must equal seal inputs, and resolver replay binds the immutable manifest back to the frozen request | three-field mismatch matrix and resolver mutation fixture |
| CP7 writer ownership | Non-blocking OS ownership lock is held on the existing per-family event ledger; same-process handles share it, independent processes fail before append | three synchronized spawn-process rounds, exactly one accepted writer each |
| CP7 terminal failure latch | Short `os.write` and raised `OSError` latch deterministic terminal reasons; later submit/seal cannot mutate the ledger | short-write and exception fixtures |
| CP7 same-process handle coherence | All live handles for a family share one process-authoritative commands/index/receipts/failure state under the family `RLock`; final close clears it | exhaustive same/same, same/conflict, distinct, close/reopen matrix |
| 10k characterization | exactly 10,000 deterministic commands; cold rebuild, v1 seal, tracemalloc peak and manifest size observations; no capacity threshold or SLA assertion | characterization test/property evidence |

## Minimal slices and local validation

1. Canonical/path/publish primitives: compiled successfully and passed golden/path/create-exclusive fixtures.
2. Recorder/rebuild/seal: passed append, idempotency, malformed-tail, semantic validation and deterministic seal fixtures.
3. Full-chain supersession: passed immutable v1 prefix replay, unique head, broken chain, explicit cycle, fork and tamper fixtures.
4. Characterization: completed exactly 10,000 commands with correctness assertions and four non-negative finite observations.

## 10k characterization evidence

Environment: Python 3.11.15; Linux 6.18.33.1 Microsoft WSL2 x86_64; CPU identifier `x86_64`; fixture hash `fc520fc49f5b68f99a57a3da88fe42e2688316d9d8d6d43c7a0cd24fa9183661`.

| Metric | Observed value |
|---|---:|
| `open_rebuild_elapsed_seconds` | 1.3285973829988507 |
| `seal_elapsed_seconds` | 4.82908684999893 |
| `peak_tracemalloc_bytes` | 17580027 |
| `manifest_bytes` | 504 |

These are single-machine synthetic observations. Peak memory is Python `tracemalloc` allocated bytes, not process RSS. There is deliberately no numeric pass threshold and no supported-capacity, SLA, production-scale, or long-search recoverability claim.

## Verification and handoff

- Packet test command: PASS (`196 passed`; S02 + unchanged S01 contract suite).
- QA-S02-001..004 native negative groups: PASS (`10 passed, 21 deselected`).
- QA-S02-005 exhaustive two-handle matrix: PASS (`4 passed, 27 deselected`).
- Cross-process ownership stress: PASS (`3 passed, 28 deselected`).
- Crash/chain/no-resume subset: PASS (`7 passed, 20 deselected`).
- Exact-10k characterization: PASS (`1 passed, 30 deselected`); four observations recorded above with unchanged fixture hash and no threshold.
- S02 module `py_compile`: PASS.
- `git diff --check`: PASS.
- Story return-check and evidence-check: PASS.
- QA focus: independently probe immutable publish failure between manifest/validation, graph error precedence, symlink containment, partial-write preservation, and the no-resume/new-family disclosure.
- Remaining product risk: a partial long-running ledger intentionally loses native continuation; the blocked family is retained and the entire search must restart from pre-search declaration under a new `family_id`. Segmented ledger/checkpoint/incremental seal/sharding remains a follow-up storage CR trigger, not part of S02.
- Documentation focus: never translate the 10k observations into capacity or readiness language.

No design delta is required; the implementation follows LLD v1.1 without changing HLD, ADR, Feature design, or the S01 public contract.

## CP7 rework round 1

Closed QA-CR163-S02-001 through QA-CR163-S02-004 without weakening the LLD or adding a persistent artifact type. The writer ownership primitive locks the existing `events.jsonl`, so the fixed six-artifact layout remains unchanged. `close()` releases ownership for a later full-ledger rebuild; it does not expose a recovery cursor, truncate/repair behavior, or in-place continuation after malformed evidence. All failure and concurrency fixtures use only synthetic local `tmp_path` roots.

## CP7 rework round 2

Closed QA-CR163-S02-005 with a process-authoritative per-family state shared by every live recorder handle. Handles no longer snapshot independent event indexes: commands, canonical event bytes, receipts, and terminal I/O failure are shared under the existing family lock, preserving average O(1) submit behavior. Identical cross-handle replay is idempotent; conflicting identity is rejected before append; distinct facts serialize as complete lines. Reference-counted close keeps the state while any handle remains live, then releases the ledger ownership and removes the process state so a clean reopen must rebuild from disk. This changes no public lineage object, artifact layout, no-resume rule, or long-lived design.
