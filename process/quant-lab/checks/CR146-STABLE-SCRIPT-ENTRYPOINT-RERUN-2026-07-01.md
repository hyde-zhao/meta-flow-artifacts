---
check_id: "CR146-STABLE-SCRIPT-ENTRYPOINT-RERUN-2026-07-01"
type: "script_entrypoint_contract"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-07-01T07:18:00+08:00"
scope:
  active_change: "CR-146"
  source_cr: "CR-139"
  real_lake_read: true
  lake_write_executed: false
  legacy_script_command_executed: false
---

# CR146 Stable Script Entrypoint Rerun

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Stable duplicate profile entrypoint exists | PASS | `scripts/data_lake/profile_duplicate_keys.py` | New stable command surface for read-only duplicate-key profiling. |
| Stable duplicate split-planning entrypoint exists | PASS | `scripts/data_lake/plan_duplicate_resolution.py` | New stable command surface for read-only duplicate-resolution planning. |
| Legacy path not required for CR146 execution command | PASS | This check | CR146 rerun commands use `scripts/data_lake/*`; legacy modules remain archived only. |

## Checklist

| 检查项 | 结果 | 证据 | 说明 |
|---|---|---|---|
| Batch0 duplicate profile rerun via stable entrypoint | PASS | `process/evidence/CR146-GATEB-BATCH0-DUPLICATE-PROFILE-2026-06-30.json` | Command: `uv run --python 3.11 python scripts/data_lake/profile_duplicate_keys.py --lake-root /home/hyde/data/quant-lab/data-lake --out process/evidence/CR146-GATEB-BATCH0-DUPLICATE-PROFILE-2026-06-30.json --sample-limit 5 --sample-rows-per-key 0`. |
| Batch2 split planning rerun via stable entrypoint | PASS | `process/evidence/CR146-GATEB-BATCH2-SPLIT-PLANNING.index.json` | Command: `uv run --python 3.11 python scripts/data_lake/plan_duplicate_resolution.py --lake-root /home/hyde/data/quant-lab/data-lake --out process/evidence/CR146-GATEB-BATCH2-SPLIT-PLANNING-2026-06-30.json --index-out process/evidence/CR146-GATEB-BATCH2-SPLIT-PLANNING.index.json --sample-limit 3 --max-source-pairs 6`. |
| Output JSON validity | PASS | `python -m json.tool` | Batch0 detail and Batch2 index JSON are valid. |
| Operation boundary | PASS | evidence `operation_counts` | provider/lake write/catalog/pointer/physical migration/NAS/runtime/credential/Git remote counts remain 0. |
| Business-conflict count retained | PASS | Batch2 index totals | `business_conflict_group_count=4,272,624`, `source_row_count=189,950,712`. |
| Stable entrypoint registry updated | PASS | `scripts/quality/check_script_entrypoints.py`, `docs/components/SCRIPT-ENTRYPOINTS.md` | The new supported data-lake entrypoints are registered. |

## Exit Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| CR146 evidence no longer depends on legacy command invocation | PASS | Evidence was regenerated through `scripts/data_lake/profile_duplicate_keys.py` and `scripts/data_lake/plan_duplicate_resolution.py`. |
| No new risk introduced | PASS | Rerun was read-only and only overwrote process evidence files. |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Stable duplicate profile entrypoint | `scripts/data_lake/profile_duplicate_keys.py` | PASS | Supported command surface. |
| Stable split planning entrypoint | `scripts/data_lake/plan_duplicate_resolution.py` | PASS | Supported command surface. |
| Updated validation report | `process/checks/CR146-CR139-POST-CLOSE-LAKE-VALIDATION-2026-06-30.md` | PASS | References this rerun check. |

## Conclusion

Result: `PASS`.

CR146 duplicate profile and split-planning evidence has been rerun through stable `scripts/data_lake/*` entrypoints. The legacy CR-specific scripts remain archive/backward-compatibility sources only and are not the CR146 execution command surface.
