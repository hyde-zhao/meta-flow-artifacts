# CR139 W3 Governance Planning

- status: `planned`
- created_at: `2026-06-30T08:04:00+08:00`
- source: CR139 W2 CP8 follow-up gates
- prior_current_truth: CR139 W2 Gate D published 17/17 datasets
- prior_cleanup: Gate F-1 executed, Gate F-2 legacy retained/superseded plan generated
- prior_nas: Gate H sync executed and post-sync dry-run verified
- prior_provider_catalog: Gate G deferred because no production provider catalog consumer is confirmed

## Goal

W3 turns the W2 current-truth migration into durable governance: reader coverage for pointer-level-only datasets, explicit retention policy for legacy runs, automated publish guards, and recurring validation that detects drift before it affects research or production users.

## Workstreams

| ID | Workstream | Scope | Priority | Exit Criteria |
|---|---|---|---|---|
| W3-S01 | Reader support expansion | Add or explicitly waive `read_dataset()` support for the 7 pointer-level-only datasets. P0: `liquidity_capacity`, `market_cap`; P1: `prices_limit_code_change_fixes`, `prices_limit_coverage_exclusions`, plus conditional `industry_classification` support; P2: `bse_code_mapping`, `lifecycle_code_change`. `industry_classification` is a non-PIT snapshot in the current lake, so default `read_dataset()` must remain blocked and `QualityPolicy(allow_warn=True)` is required to read it with explicit `non_pit_snapshot` issues. Before implementation, run a read-only reader API extension assessment: locate the supported dataset registration path, schema compatibility contract, row-count/smoke strategy, and performance risk for the two P0 17M-row datasets. | P0/P1/P2 | Reader API extension assessment PASS; P0 datasets have explicit implementation scope and schema/performance smoke plan; reader smoke covers every dataset that has a real consumer; waived datasets have documented pointer-level-only rationale; conditional readers must record default status, allow-warn status, and row-count evidence. |
| W3-S02 | Legacy retention and superseded register | Keep 210 legacy canonical candidates by default, mark them superseded, and define archive/delete eligibility only after dependency review. | P0 | A machine-readable superseded register exists; delete remains blocked unless dependency proof is 0 and rollback/archive plan exists. |
| W3-S03 | Publish guard automation | Convert W2 Gate D/Gate C checks into reusable guards: PIT purity, candidate-path leak=0, lineage checksum valid, catalog/manifest hash before-after, row-count parity, reader support matrix, NAS/provider not implicit. | P0 | A single guard command can fail-closed before pointer advance or active catalog refresh. |
| W3-S04 | Recurring validation | Schedule or document repeatable no-write validation for active catalog pointers, manifest linkage, row counts, PIT flags, quality quarantine references, NAS drift dry-run, and provider catalog deferred status. | P1 | Validation pack is repeatable and produces stable PASS/WARN/BLOCKED outputs. |
| W3-S05 | Operational config hygiene | Resolve stale local config discovered by Gate H and verify all four data-lake root variables: `MARKET_DATA_LAKE_ROOT`, `MARKET_DATA_LAKE_ARCHIVE_ROOT`, `MARKET_DATA_LAKE_BACKUP_ROOT`, `MARKET_DATA_LAKE_RESTORE_ROOT`. Compare `.env` key presence/paths to `.env.example` without printing secrets. | P0 | All 4 roots point to data-lake-aligned paths; `.env` and `.env.example` are key-consistent for lake/archive/backup/restore; guarded wrappers fail closed on stale config and never print secrets. |

## Release Slices

| Slice | Included Workstreams | User Value | Gate |
|---|---|---|---|
| W3-A reader P0 + config hygiene | W3-S05, W3-S01 read-only extension assessment, W3-S01 P0 | Research users can consume `liquidity_capacity` and `market_cap`; operations no longer trip over stale lake root config. | CP5/CP7 scoped implementation |
| W3-B retention + publish guard | W3-S02, W3-S03 | Future publish/catalog changes fail closed and historical runs remain traceable. | CP5/CP7 scoped implementation |
| W3-C recurring validation + reader P1/P2 | W3-S01 P1/P2, W3-S04 | Current truth remains continuously verifiable; pointer-only waivers are explicit; non-PIT conditional reader support is visible rather than silently counted as default available. | CP5/CP7 scoped implementation |
| W3-D provider catalog re-evaluation | Gate G deferred record, production consumer discovery | Provider catalog remains deferred unless a real production consumer appears; if one appears, Gate G becomes a separate explicit write gate. | Decision-only / separate Gate G |

## Backlog And Decisions

| ID | Type | Item | Default |
|---|---|---|---|
| W3-DQ-001 | scope | Which pointer-level-only datasets need full reader support? | Implement P0, review P1, waive P2 unless consumer appears. |
| W3-DQ-002 | risk_acceptance | Can any legacy canonical run be deleted? | No. Default retain + superseded until dependency proof and archive/rollback plan exist. |
| W3-DQ-003 | runtime_authorization | Should recurring validation include NAS dry-run? | Yes, dry-run only; no sync/delete without explicit gate. |
| W3-DQ-004 | implementation | Should `.env` be edited or should wrappers override stale config? | Prefer explicit config correction after secret-safe review; wrappers continue fail-closed. The review must cover all 4 root variables and `.env.example` key consistency. |
| W3-DQ-005 | implementation | Should W3-A implement reader P0 immediately or first run reader API extension assessment? | Run read-only extension assessment first, then implement P0 support only after registration/schema/performance scope is known. |

## Not Authorized By This Plan

- Provider catalog/provider-lake-catalog write.
- NAS sync/delete.
- Legacy canonical delete/archive.
- Active catalog or manifest mutation.
- Runtime/trading/Git remote operation.
- Any credential disclosure.
