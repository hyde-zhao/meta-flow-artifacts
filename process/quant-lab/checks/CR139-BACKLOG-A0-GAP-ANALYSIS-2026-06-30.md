---
checkpoint_id: "CR139-BACKLOG-A0-GAP-ANALYSIS-2026-06-30"
checkpoint_name: "CR139 Backlog-A0 Gap Analysis"
type: "read_only_gap_analysis"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-30T10:14:26+08:00"
checked_at: "2026-06-30T10:14:26+08:00"
scope:
  active_change: "CR-139"
  stories:
    - "CR139-S23"
    - "CR139-S27"
    - "CR139-S28"
    - "CR139-S29"
    - "CR139-S38"
  side_check:
    - "CR139-S25-vs-CR139-S28-overlap"
  no_lake_write: true
  no_active_catalog_write: true
  no_active_manifest_append: true
  no_pointer_advance: true
  no_nas_provider_runtime_git: true
---

# CR139 Backlog-A0 Gap Analysis

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Process route healthy | PASS | `meta-flow workspace check` | `process` symlink healthy; routing mode is `symlink`. |
| Resume scope bounded | PASS | `process/handoffs/CR139-BACKLOG-A0-RESUME-HANDOFF-2026-06-30.md` | Scope limited to read-only gap analysis for S23/S27/S28/S29/S38 plus S25-vs-S28 overlap. |
| W2/W3 not rerun | PASS | `process/STORY-STATUS-CR139.md`, `process/checks/CR139-40-STORY-RECONCILIATION-2026-06-30.md` | W2 current truth and W3 governance are treated as existing evidence only. |
| Story baseline readable | PASS | `process/stories/CR139-S23-*.md`, `CR139-S27-*.md`, `CR139-S28-*.md`, `CR139-S29-*.md`, `CR139-S38-*.md`, `CR139-S25-*.md` | Original scope and lld_policy were read from Story cards. |
| Forbidden operations absent | PASS | This file only | No lake/catalog/manifest/pointer/NAS/provider/runtime/Git operation was executed. |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | Original scope summarized for S23/S27/S28/S29/S38 | PASS | Story cards + `process/STORY-BACKLOG-CR139.md` | See per-Story table below. |
| 2 | W2/W3 Gate evidence mapped to each Story | PASS | Gate B/D/W3 checks and reconciliation check | Evidence is partial for all five target stories; none can close by existing Gate evidence alone. |
| 3 | Exact gap classification assigned | PASS | Per-Story analysis | All five target stories remain `needs_full_story_execution`; existing Gate evidence narrows but does not replace CP5/CP6/CP7. |
| 4 | CP5 scope recommendation assigned | PASS | Story lld_policy + evidence gap | S23/S27/S29 remain `full-lld`; S28/S38 remain `technical-note`. |
| 5 | Backlog-A dependency order produced | PASS | `process/DEVELOPMENT-PLAN-CR139.yaml` and file ownership matrix | S29 + S23 first remains the minimal executable package. |
| 6 | S25-vs-S28 overlap checked | PASS | Story cards, `readers.py`, `readiness.py`, development plan merge order | Keep S25 independent; define shared helper boundary to avoid duplicate gates. |

## Evidence Read Set

| Evidence | Path | Relevant finding |
|---|---|---|
| Story reconciliation | `process/checks/CR139-40-STORY-RECONCILIATION-2026-06-30.md` | S23/S27/S28/S29/S38 are explicitly listed as partial Gate coverage only and true backlog. |
| Story status | `process/STORY-STATUS-CR139.md` | All five target stories remain `lld-pending`; S25 also remains `lld-pending`. |
| Gate B adj_factor write | `process/checks/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION-2026-06-29.md` | `adj_factor` candidate write verified, duplicate keys removed, but only candidate-layer data hygiene was in scope. |
| Gate B integrated review | `process/checks/CR139-W2-GATEB-BATCH2-INTEGRATED-EVIDENCE-REVIEW-2026-06-29.md` | Batch2 candidate layer complete; no Story-level PIT/read gate tests authorized by this review. |
| Gate D post-pointer smoke | `process/checks/CR139-W2-GATED-POST-POINTER-SMOKE-2026-06-29.md` | 17 current published pointers, 4 reader smokes, 2 PIT-clean datasets; not a substitute for S23/S27/S28/S29 acceptance. |
| W3 publish guard | `process/checks/CR139-W3B-PUBLISH-GUARD-2026-06-30.md` | Current truth is publish-safe; `adj_factor` is reader-guard-pass but PIT applicability is false in this guard. |
| W3 reader P0 | `process/checks/CR139-W3A-READER-P0-IMPLEMENTATION-2026-06-30.md` | P0 reader registration for liquidity/market_cap only; not read-audit or readiness pre-read gate. |
| W3 recurring validation | `process/checks/CR139-W3C-RECURRING-VALIDATION-2026-06-30.md` | Recurring pack passes with zero writes, but only validates existing guard scripts and reader support matrix. |
| Existing code hints | `market_data/readers.py`, `market_data/readiness.py`, `market_data/normalization.py` | Existing PIT/as-of, availability, adjustment, and readiness primitives exist; target Story artifacts are not complete. |

## Per-Story Gap Classification

| Story | Original scope | Existing W2/W3 coverage | Gap classification | CP5 recommendation | Rationale |
|---|---|---|---|---|---|
| S23 `adjustment-factor-pit` | REQ-229: 除权除息事件按 PIT 应用；复权因子时点正确；复权断点回归通过. Output slice: `market_data/normalization.py`. | Gate B wrote `adj_factor` candidate, removed duplicate keys, and W3 publish guard sees current `adj_factor` rows. `normalization.py` has adjustment factor normalization and `available_at` fields. | `needs_full_story_execution` | `full-lld` | Existing evidence proves data hygiene/current publication, not PIT application of ex-dividend events or breakpoint regression. W3 PIT guard marks `adj_factor` PIT applicability false, so it cannot close REQ-229. |
| S27 `read-audit-log-runid` | REQ-233: reader read audit log is recorded and connected to `RunEvidenceIndex` by the same run-id. Outputs: `market_data/readers.py`, new `market_data/read_audit.py`. | S22/Gate C/D establish write-side lineage and run-id current-truth propagation. `trading/strategy_runner/evidence_index.py` has `RunEvidenceIndex`. No `market_data/read_audit.py` exists and no read-audit hook evidence was found. | `needs_full_story_execution` | `full-lld` | Run-id producer evidence is not reader audit evidence. This Story still needs a read-side audit contract, hook boundary, fixture tests, and CP6/CP7 proof that the reader log links to `RunEvidenceIndex`. |
| S28 `readiness-pre-read-gate` | REQ-234: `build_readiness_matrix` becomes a mandatory pre-read gate for coverage/freshness/PIT; failed readiness blocks reads. Outputs: `market_data/readiness.py`, `market_data/readers.py`. | `build_readiness_matrix` exists and W3 publish guard/recurring validation consume readiness metadata. Reader P0 implementation covers liquidity/market_cap registration. | `needs_full_story_execution` | `technical-note` | Existing readiness is a matrix/guard capability, not proven as a mandatory reader pre-read gate. CP5 should narrow this to the read-path hook, failure codes, and fixture proof while preserving the existing readiness matrix contract. |
| S29 `pit-dedup-correctness-tests` | REQ-226/227: future financial report fixture must be invisible to as-of reader; duplicate-key fixture is unique after dedup read. Outputs: `tests/test_cr139_pit_correctness.py`, `tests/test_cr139_dedup_correctness.py`. | Existing W1 tests cover `read_panel_as_of` PIT filtering and read-layer dedup separately (`tests/test_cr139_pit_as_of_reader.py`, `tests/test_cr139_read_layer_dedup.py`). Gate B/D/W3 prove current-data hygiene. | `needs_full_story_execution` | `full-lld` | Existing tests are useful predecessors but not the explicit S29 regression suite and not the combined future-financial-report + duplicate-key acceptance. S29 should define the correctness standard before S23/S28 rely on it. |
| S38 `quality-partition-reorg` | REQ-210: `quality/` is partitioned by dataset/date; smoke/probe output is isolated under `_scratch/`. Output: `market_data/quality_writer.py`. | W3 publish guard confirms quality paths exist; Gate B/D evidence has quarantine quality paths. W3 retention register covers superseded/retention governance. No `market_data/quality_writer.py` file exists. | `needs_full_story_execution` | `technical-note` | Existing quality paths are evidence artifacts, not a reusable writer/reorg contract. The `_scratch/` smoke/probe isolation and dataset/date partition writer still need design and verification. |

## Existing Evidence That Must Not Be Overstated

| Evidence | Can be reused for | Cannot be used to claim |
|---|---|---|
| `adj_factor` candidate/current publication | S23 fixtures and baseline row-count/source-truth references | PIT ex-dividend application, factor timing correctness, or breakpoint regression closure |
| S05 PIT reader and S07 dedup tests | S29 predecessor contracts and fixture patterns | S29 full acceptance closure |
| S22/Gate C/D lineage and run-id proof | S27 run-id source contract | S27 read-audit log hook or read-side audit persistence |
| `build_readiness_matrix` + W3 publish guard | S28 technical-note input and pre-read gate predicates | Mandatory reader gate integration |
| W3 `quality_paths_exist` and retention register | S38 inventory of existing path shapes | `quality_writer.py` partition contract or `_scratch/` isolation |

## S25 vs S28 Overlap Decision

Decision: `keep independent`.

| Candidate | Decision | Reason |
|---|---|---|
| Merge S25 into S28 | Rejected | S25 covers `decision_time` vs `available_at` lookahead semantics in the reader path. S28 covers dataset/readiness preconditions such as coverage, freshness, and PIT readiness. Merging would blur separate REQ-235 and REQ-234 acceptance. |
| Keep independent | Selected | The development plan already gives S25 and S28 separate REQs, lld_policy, and readers.py merge slices. S25 is the semantic lookahead gate; S28 is the readiness gate that may call or compose semantic predicates. |
| Downgrade S25 to test/guard item | Rejected for now | Existing `read_panel_as_of` and corporate-action checks partially cover available-at filtering, but S25's explicit signal-time lookahead gate is still a backlog Story with its own acceptance criterion. |
| Defer with reason | Partially applicable | S25 can stay outside Backlog-A0 executable package, but S28 CP5 must reserve the integration point so later S25 does not rework S28's reader hook. |

Boundary rule for CP5:

- S25 owns the reusable decision-time/lookahead predicate and failure semantics.
- S28 owns the pre-read readiness orchestration and may invoke S25's predicate once S25 exists.
- If S28 executes before S25, S28 must leave an explicit hook/extension point and must not silently implement only part of S25 under a different name.

## Backlog-A Dependency Order

| Order | Package | Stories | Why |
|---:|---|---|---|
| 1 | Backlog-A1 | S29 | Defines the PIT/dedup correctness standard and regression fixture pattern before downstream stories rely on it. |
| 2 | Backlog-A2 | S23 | Uses S05 PIT semantics and should consume S29's correctness standard for adjustment-factor PIT/breakpoint tests. |
| 3 | Backlog-A3-design | S27 + S28 | Both are read-path gates and can be designed together to avoid duplicate hooks. Development must respect `readers.py` merge order and S25 boundary. |
| 4 | Backlog-A4 | S38 | Can be designed independently, but implementation should wait until any quality path touchpoints are clear; no dependency on S29/S23. |
| Side boundary | Backlog-E / dependency note | S25 | Keep independent from S28; schedule before or alongside S28 only if the next package chooses to touch the same `readers.py` gate surface. |

Minimal next executable package: `S29 + S23`.

Reason: A0 found no valid direct closure by existing Gate evidence. S29 gives the PIT/dedup regression standard; S23 is the highest-value data consistency story with strong existing data evidence but missing PIT correctness proof. This package remains process/planning only until CP5 is created and approved for those stories.

## CP5 Execution Notes For Backlog-A1/A2

| Item | Requirement | Reason |
|---|---|---|
| S29 standard before S23 reference | S29 LLD must define the correctness standard section first, including fixture mode, future-financial-report assertion, duplicate-key assertion, and expected failure/status semantics. S23 LLD must explicitly reference that section. | Prevents parallel CP5 drafting from producing inconsistent PIT/dedup acceptance criteria. |
| S29/S23 consistency check | CP5 precheck must verify that S23 adjustment-factor PIT acceptance cites the S29 correctness standard and does not introduce a conflicting PIT/as-of or dedup rule. | S23 uses S29 as the validation standard; mismatch would make CP6/CP7 ambiguous. |
| S29 test boundary | CP5 design must show that `tests/test_cr139_pit_correctness.py` and `tests/test_cr139_dedup_correctness.py` do not duplicate or conflict with existing `tests/test_cr139_pit_as_of_reader.py` and `tests/test_cr139_read_layer_dedup.py`. | Existing W1 tests are predecessor contracts; S29 adds explicit regression acceptance, not a rewrite of W1 coverage. |
| S23 write-path protection | CP5 design must state that the `market_data/normalization.py` adjustment-factor slice preserves the Gate B-verified `adj_factor` write path and does not mutate lake/catalog/manifest/pointer surfaces. | Gate B already verified candidate/current data hygiene; S23 should add PIT correctness logic/tests without invalidating that baseline. |
| File owner validation | CP5 precheck must verify no file-owner conflict: S29 owns new tests; S23 owns `market_data/normalization.py` adjustment-factor slice plus any S23-specific tests. | Confirms the first package can proceed without mixing S27/S28 readers.py or S38 quality path work. |

## Exit Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| No backlog Story silently closed | PASS | S23/S27/S28/S29/S38 remain backlog; this file does not change Story status. |
| Gap classification complete | PASS | Each Story has exact classification and CP5 recommendation. |
| S25 overlap resolved for planning | PASS | Keep independent, with explicit CP5 boundary and integration sequencing. |
| No manual gate required | PASS | No risk acceptance or runtime/write authorization is requested by this read-only analysis. |
| No forbidden operation executed | PASS | Operation counts are all zero for lake/catalog/manifest/pointer/NAS/provider/runtime/Git. |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Backlog-A0 gap analysis | `process/checks/CR139-BACKLOG-A0-GAP-ANALYSIS-2026-06-30.md` | PASS | This file. |
| STATE update | N/A | N/A | No Story/gate/status change occurred, so `process/STATE.md` and `process/state/STATE.current.json` were intentionally not updated. |

## Conclusion

- Result: `PASS`
- Exact classifications:
  - S23: `needs_full_story_execution`, CP5 `full-lld`
  - S27: `needs_full_story_execution`, CP5 `full-lld`
  - S28: `needs_full_story_execution`, CP5 `technical-note`
  - S29: `needs_full_story_execution`, CP5 `full-lld`
  - S38: `needs_full_story_execution`, CP5 `technical-note`
- S25-vs-S28 overlap decision: `keep independent`
- Minimal next executable package: `S29 + S23`
- Operation boundary: read-only process analysis; no lake/catalog/manifest/pointer/NAS/provider/runtime/Git operation executed.
