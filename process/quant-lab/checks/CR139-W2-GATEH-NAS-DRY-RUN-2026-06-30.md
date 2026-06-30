# CR139 W2 Gate H NAS Dry-Run

- status: `pass_gate_h_nas_dry_run`
- evidence: `/home/hyde/workspace/quant-lab/process/evidence/CR139-W2-GATEH-NAS-DRY-RUN-2026-06-30.json`
- command_returncode: `0`
- active_catalog_before_after: `0ac6c1b8da8b7624f3d0dfbf879f1741254284d1faa03d42a1e01e4188c5c989` / `0ac6c1b8da8b7624f3d0dfbf879f1741254284d1faa03d42a1e01e4188c5c989`
- active_manifest_before_after: `32887fd7989959ff7e09daba702ea12262b6fba1837a56d1a800e52248ef54d7` / `32887fd7989959ff7e09daba702ea12262b6fba1837a56d1a800e52248ef54d7`
- local_lake_files_before_after: `18592` / `18592`
- local_lake_bytes_before_after: `26436899912` / `26436899912`

## Boundary

- direction: push
- scope: lake only
- execute: false
- delete: false
- no active catalog write
- no active manifest append
- no provider catalog write
- no provider-lake-catalog write
- no runtime/trading/Git remote
- NAS credentials were used only by rsync authentication and were not printed or persisted.

## Checks

| Check | Result |
|---|---|
| `command_returned_zero` | PASS |
| `dry_run_mode_confirmed` | PASS |
| `delete_disabled_confirmed` | PASS |
| `scope_lake_only_confirmed` | PASS |
| `active_catalog_unchanged` | PASS |
| `active_manifest_unchanged` | PASS |
| `local_lake_tree_unchanged` | PASS |
| `no_secret_markers_in_output` | PASS |

## RSync Stats

| Field | Value |
|---|---|
| Number of files | 30,298 (reg: 18,592, dir: 11,706) |
| Number of created files | 129 (reg: 63, dir: 66) |
| Number of deleted files | 0 |
| Number of regular files transferred | 65 |
| Total file size | 26.44G bytes |
| Total transferred file size | 2.93G bytes |
| Literal data | 0 bytes |
| Matched data | 0 bytes |
| File list size | 458.74K |
| File list generation time | 0.001 seconds |
| File list transfer time | 0.000 seconds |
| Total bytes sent | 1.50M |
| Total bytes received | 12.60K |

## Decision

- Dry-run PASS unlocks Gate H sync authorization review only.
- This dry-run does not authorize NAS write/sync execution.
