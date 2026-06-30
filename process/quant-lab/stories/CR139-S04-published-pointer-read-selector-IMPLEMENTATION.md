---
story_id: "CR139-S04"
change_id: "CR-139"
checkpoint: "CP6"
status: "implemented"
implemented_at: "2026-06-28T21:22:52+08:00"
implemented_by: "host-orchestrator-inline"
dispatch_ref: "ADE-CR139-W1-META-DEV-INLINE-S04-COMPLETE-2026-06-28T212252+0800"
context_ref: "process/context/stories/STORY-CR139-S04.CP6.work-packet.json"
---

# CR139-S04 Implementation — V1 Published Pointer / Read Selector

## Implementation Objects

| Object | Path | Status | Notes |
|---|---|---|---|
| Published-only selector | `market_data/catalog.py` | done | Added `CATALOG_CURRENT_POINTER_NOT_PUBLISHED` and `CatalogStore.get_published_current_pointer()` while keeping legacy `get_current_pointer()` unchanged |
| Fixture tests | `tests/test_cr139_published_pointer.py` | done | Covers unpublished block, published pointer return, validate no auto-publish, dry-run publish no catalog write, reader published-only behavior |
| Existing explicit publish gate | `market_data/publish.py` | reused | `publish_current_pointer()` already dry-run only by default and blocks real publish with `REAL_PUBLISH_NOT_AUTHORIZED` |
| Existing CLI contract | `market_data/cli.py` | reused | `p0-publish` / `p0-read` already expose fixture-safe dry-run publish/read contracts; legacy `publish` command was not changed |

## Contract Mapping

| Design Contract | Implementation | Verification |
|---|---|---|
| Read selector must fail closed for candidate/unpublished entries | `CatalogStore.get_published_current_pointer()` raises `CatalogError(CATALOG_CURRENT_POINTER_NOT_PUBLISHED)` before pointer conversion | `test_published_current_pointer_selector_blocks_unpublished_candidate` |
| Published entry exposes current pointer | published catalog entry converts through `catalog_pointer_from_entry()` | `test_published_current_pointer_selector_returns_only_published_entry` |
| Validate pass must not advance pointer | `validate_p0_candidate()` keeps `publish_count=0` and `current_pointer_changes=0`; catalog tree hash unchanged | `test_validate_pass_does_not_publish_or_advance_current_pointer` |
| Explicit publish gate remains dry-run only in this scope | `publish_current_pointer(..., dry_run=True)` reports one planned pointer change with zero catalog / lake writes | `test_explicit_publish_gate_dry_run_does_not_write_catalog_current_pointer` |
| Reader consumes only published catalog entries | existing `read_dataset()` `entry.published` gate blocks candidate, then reads after fixture publish | `test_reader_only_reads_published_catalog_entry` |

## Validation

Command:

```bash
uv run --python 3.11 pytest -q tests/test_cr139_published_pointer.py
```

Result: `5 passed in 0.49s`.

Regression command:

```bash
uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py tests/test_cr139_s02_golden_baseline.py tests/test_cr139_duplicate_fingerprint_profiling.py tests/test_cr139_published_pointer.py tests/test_cr014_catalog_publish_gate.py tests/test_cr014_p0_pipeline_contract.py
```

Result: `46 passed in 2.15s`.

## Deviations

No behavioral deviation from the S04 technical note. The implementation did not change legacy `cmd_publish()` because existing tests and legacy users depend on that command's catalog write behavior. S04 instead adds a new explicit published-only selector and verifies the existing CR014 dry-run publish gate and P0 read contracts.

## Remaining Risks

| Risk | Status | Route |
|---|---|---|
| Real pointer advance / production catalog publish | not-authorized | Remains out of scope; only fixture catalog writes and dry-run gate validation were executed |
| Legacy `cmd_publish()` still writes fixture/local catalog when called | known-existing-behavior | Not changed to avoid breaking pre-CR139 callers; S04 consumers should use `get_published_current_pointer()` / P0 dry-run gate contracts |
