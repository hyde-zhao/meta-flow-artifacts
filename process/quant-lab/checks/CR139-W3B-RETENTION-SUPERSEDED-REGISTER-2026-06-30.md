# CR139 W3-B Retention Superseded Register

## Result

- result: `pass_w3b_retention_superseded_register`
- register: `process/registers/CR139-W3-LEGACY-SUPERSEDED-REGISTER-2026-06-30.json`
- register_sha256: `e1082469f3aa925f3e0ca1cf824dc5f1ea4d82fd6a5296a1a404c9359997765f`

## Summary

| metric | value |
|---|---:|
| legacy_record_count | 210 |
| dataset_count | 17 |
| total_size_bytes | 4346487187 |
| total_row_count | 156477393 |

## Checks

| check | result |
|---|---:|
| gate_f2_source_pass | True |
| legacy_record_count_210 | True |
| all_records_retain | True |
| archive_delete_not_authorized | True |
| legacy_paths_exist | True |

## Operation Counts

```json
{
  "active_catalog_write": 0,
  "active_manifest_append": 0,
  "credential_read": 0,
  "git_remote": 0,
  "lake_archive": 0,
  "lake_delete": 0,
  "nas_operation": 0,
  "provider_catalog_write": 0,
  "runtime_operation": 0
}
```
