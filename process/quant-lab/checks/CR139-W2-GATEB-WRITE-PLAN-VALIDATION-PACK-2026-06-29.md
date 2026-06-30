# CR139 W2 Gate B Write Plan Validation Pack

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Write plan exists | PASS | `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.json`, `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.md` | Machine and human plans exist. |
| Batch0 decision approved for plan generation | PASS | `process/checkpoints/CR139-W2-GATEB-BATCH0-DECISION-BRIEF.md` | Approval authorizes write-plan generation only. |
| Gate A/B0 evidence exists | PASS | `process/evidence/CR139-W2-GATEA-REAL-LAKE-INVENTORY-DATALAKE-2026-06-29.json`, `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE.index.json` | Inputs available for validation. |

## Checklist

| # | Validation Item | Status | Evidence | Result / Handling |
|---:|---|---|---|---|
| 1 | Plan JSON/Markdown dataset row consistency | PASS | Consolidated Python validation | JSON dataset count = 17, MD dataset rows = 17, dataset names and run IDs match. |
| 2 | 17 datasets match Gate A inventory | PASS | Consolidated Python validation | `dataset_set_matches_inventory=True`. |
| 3 | Planned run IDs rebuild and validate | PASS | Consolidated Python validation | `run_id_rebuild_match=17/17`, `all_run_ids_valid=True`. |
| 4 | Legacy catalog paths match inventory | PASS | Consolidated Python validation | `path_mismatch_count=0`. |
| 5 | Batch grouping is consistent | PASS | Consolidated Python validation | Batch 0=7, Batch 1A=8, Batch 1B=2, Batch 2=7. |
| 6 | Batch 1A candidate conditions | PASS | Consolidated Python validation | All Batch 1A datasets have `duplicate_key_count=0`, `published=false`, and `row_count<=100000`. |
| 7 | Quarantine path isolation | PASS | Consolidated Python validation | Batch 2 quarantine paths are under the new candidate run-id path, do not equal candidate root, and do not point to legacy canonical paths. |
| 8 | Brief hard requirements trace | PASS | Consolidated Python validation | 8/8 write-plan hard requirements mapped to plan fields. |
| 9 | Deterministic dedup sort policy | PASS | Consolidated Python validation | `dedup_sort_policy_present=True`, Batch 2 sort-key coverage 7/7. |
| 10 | Operation counts remain zero | PASS | Consolidated Python validation | provider_fetch=0, lake_write=0, catalog_write=0, pointer=0, migration=0, credential=0, NAS=0, runtime=0, Git remote=0. |
| 11 | Unauthorized boundaries preserved | PASS | Consolidated Python validation | Real lake write, catalog write, pointer advance, physical migration, credential read, NAS/runtime, and Git remote write remain not authorized. |
| 12 | State check | PASS | `uv run meta-flow state check --project-root .` | State v2 Check: OK. |
| 13 | Workspace check | PASS | `uv run meta-flow workspace check --project-root .` | process link health OK. |
| 14 | Gate event ledger check | PASS | `uv run meta-flow event check --ledger process/state/GATE-LEDGER.ndjson --type gate` | Event Ledger Check: OK. |
| 15 | Diff whitespace check | PASS | `git diff --check` | No whitespace errors. |

## Consolidated Validation Command

```bash
uv run --python 3.11 python - <<'PY'
import json
import re
from pathlib import Path
from market_data.lake_layout import build_cr139_run_id, validate_cr139_run_id

plan_path = Path('process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.json')
md_path = Path('process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.md')
inv_path = Path('process/evidence/CR139-W2-GATEA-REAL-LAKE-INVENTORY-DATALAKE-2026-06-29.json')
brief_path = Path('process/checkpoints/CR139-W2-GATEB-BATCH0-DECISION-BRIEF.md')
idx_path = Path('process/evidence/CR139-W2-GATEB-WRITE-PLAN.index.json')

plan = json.loads(plan_path.read_text())
inv = json.loads(inv_path.read_text())
idx = json.loads(idx_path.read_text())
md = md_path.read_text()
brief = brief_path.read_text()

datasets = plan['datasets']
inv_entries = inv['entries']
plan_names = {d['dataset'] for d in datasets}
inv_names = {e['dataset'] for e in inv_entries}
inv_by_dataset = {e['dataset']: e for e in inv_entries}
batch_counts = {b['batch']: len(b['datasets']) for b in plan['batching']}
batch1a = [d for d in datasets if d['batch'] == 'Batch 1A']
batch2 = [d for d in datasets if d['batch'] == 'Batch 2']
rebuilt = {
    d['dataset']: build_cr139_run_id(
        dataset=d['dataset'], source='legacy_lake', as_of_date='2026-06-29', purpose='canonical'
    )
    for d in datasets
}
md_dataset_rows = [line for line in md.splitlines() if line.startswith('| `') and 'cr139-w2-' in line]
md_dataset_names = set(re.findall(r'^\| `([^`]+)`', '\n'.join(md_dataset_rows), flags=re.M))
md_run_ids = set(re.findall(r'`(cr139-w2-[^`]+)`', '\n'.join(md_dataset_rows)))
json_run_ids = {d['planned_run_id'] for d in datasets}

hard_requirements = {
    'HR1_plan_only_no_write': all(v == 0 for v in plan['operation_counts'].values()) and 'real_lake_write' in plan['authorization_scope']['not_authorized'],
    'HR2_dataset_profiles_schema': 'dataset_profiles' in Path('process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE.index.json').read_text(),
    'HR3_deterministic_dedup_sort_key': bool(plan.get('dedup_sort_policy', {}).get('dedup_sort_key')) and all('dedup_sort_key' in d for d in batch2),
    'HR4_events_readiness_rows': any(d['dataset'] == 'events' and d.get('events_usable_canonical_rows_after_quarantine') == 351527 for d in datasets),
    'HR5_full_row_profile_precondition': any(item['id'] == 'GB-PRE-002' and item['status'] == 'not_satisfied_for_write' for item in plan['global_preconditions_before_any_write']),
    'HR6_quarantine_shape': all((d.get('quarantine_relative_path') or '').endswith('/quarantine') for d in batch2),
    'HR7_fence_precondition': any(item['id'] == 'GB-PRE-001' and item['status'] == 'not_satisfied_for_write' for item in plan['global_preconditions_before_any_write']),
    'HR8_gate_cde_plan_only': set(plan['gate_c_gate_d_gate_e_plan_only'].keys()) == {'gate_c_checksum_backfill', 'gate_d_publish_pointer', 'gate_e_path_migration'},
}

quarantine_isolated = True
for d in batch2:
    q = d.get('quarantine_relative_path') or ''
    c = d.get('candidate_relative_path') or ''
    legacy = d.get('legacy_catalog_path') or ''
    if not (q.startswith(c + '/') and q.endswith('/quarantine') and q != c and not q.startswith('canonical/') and q != legacy):
        quarantine_isolated = False

assertions = {
    'plan_json_md_dataset_rows_match': len(datasets) == len(md_dataset_rows) == 17 and md_dataset_names == plan_names and md_run_ids == json_run_ids,
    'dataset_set_matches_inventory': plan_names == inv_names,
    'run_id_rebuild_match_17_of_17': sum(d['planned_run_id'] == rebuilt[d['dataset']] for d in datasets) == 17,
    'all_run_ids_valid': all(validate_cr139_run_id(d['planned_run_id']) for d in datasets),
    'legacy_catalog_path_mismatch_count_zero': sum(d['legacy_catalog_path'] != inv_by_dataset[d['dataset']].get('canonical_path_catalog') for d in datasets) == 0,
    'batch_group_counts_match': batch_counts == {'Batch 0': 7, 'Batch 1A': 8, 'Batch 1B': 2, 'Batch 2': 7},
    'batch1a_candidate_conditions': all(d['duplicate_key_count'] == 0 and d['published'] is False and d['row_count'] <= 100000 for d in batch1a),
    'quarantine_candidate_path_isolated': quarantine_isolated,
    'brief_hard_requirements_trace_8_of_8': all(hard_requirements.values()),
    'operation_counts_all_zero': all(v == 0 for v in plan['operation_counts'].values()),
    'unauthorized_boundaries_preserved': all(k in plan['authorization_scope']['not_authorized'] for k in ['real_lake_write','catalog_write','published_pointer_advance','physical_partition_migration','credential_read','nas_runtime_or_sync','git_remote_write']),
    'machine_validation_index_present': idx.get('machine_validation', {}).get('path_mismatch_count') == 0,
}

print(json.dumps({
    'assertions': assertions,
    'hard_requirements': hard_requirements,
    'batch_counts': batch_counts,
    'md_dataset_rows': len(md_dataset_rows),
    'json_dataset_count': len(datasets),
    'run_id_rebuild_match': f"{sum(d['planned_run_id'] == rebuilt[d['dataset']] for d in datasets)}/{len(datasets)}",
    'operation_counts': plan['operation_counts'],
    'decision': 'PASS' if all(assertions.values()) else 'FAIL',
}, indent=2, ensure_ascii=False))
PY
```

## Consolidated Result

```json
{
  "decision": "PASS",
  "assertions": {
    "plan_json_md_dataset_rows_match": true,
    "dataset_set_matches_inventory": true,
    "run_id_rebuild_match_17_of_17": true,
    "all_run_ids_valid": true,
    "legacy_catalog_path_mismatch_count_zero": true,
    "batch_group_counts_match": true,
    "batch1a_candidate_conditions": true,
    "quarantine_candidate_path_isolated": true,
    "brief_hard_requirements_trace_8_of_8": true,
    "operation_counts_all_zero": true,
    "unauthorized_boundaries_preserved": true,
    "machine_validation_index_present": true
  },
  "hard_requirements": {
    "HR1_plan_only_no_write": true,
    "HR2_dataset_profiles_schema": true,
    "HR3_deterministic_dedup_sort_key": true,
    "HR4_events_readiness_rows": true,
    "HR5_full_row_profile_precondition": true,
    "HR6_quarantine_shape": true,
    "HR7_fence_precondition": true,
    "HR8_gate_cde_plan_only": true
  },
  "batch_counts": {
    "Batch 0": 7,
    "Batch 1A": 8,
    "Batch 1B": 2,
    "Batch 2": 7
  },
  "run_id_rebuild_match": "17/17"
}
```

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Validation pack complete | PASS | This file | 15 validation items covered. |
| Plan ready as future authorization basis | PASS | Checklist | Plan/check/evidence are internally consistent. |
| Write authorization unlocked | FAIL / BLOCKED | GB-PRE-001, GB-PRE-002 | Validation PASS does not authorize lake writes. Fence/Gate A rerun and target dataset full-row profile are still required. |
| Next stage boundary clear | PASS | Exit criteria | Next allowed step is pre-write readiness authorization preparation, not data lake write. |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Validation pack | `process/checks/CR139-W2-GATEB-WRITE-PLAN-VALIDATION-PACK-2026-06-29.md` | PASS | This file. |
| Write plan JSON | `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.json` | PASS | Validated. |
| Write plan Markdown | `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.md` | PASS | Validated. |
| Write plan evidence index | `process/evidence/CR139-W2-GATEB-WRITE-PLAN.index.json` | PASS | Machine validation included. |

## 结论

- 结论：`PASS_FOR_PRE_WRITE_READINESS_PREPARATION_ONLY`
- 阻断项：GB-PRE-001 fence + Gate A rerun; GB-PRE-002 target dataset full-row exact duplicate profile.
- 豁免项：None.
- 下一步：Prepare a separate pre-write readiness authorization for a named dataset, recommended `bse_code_mapping`. This does not authorize real lake writes.
- Still not authorized: real lake write, catalog/provider-lake-catalog write, published pointer advance, physical partition migration, credential read, NAS/QMT/MiniQMT/gateway runtime, trading/small_live/live, Git remote write.
