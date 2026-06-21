---
check_id: "CR102-NAS-REAL-PACKAGE-EXCHANGE-EVIDENCE"
check_name: "CR102 NAS Real Package Exchange Evidence"
type: "runtime-evidence-redacted"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-21T08:50:10+08:00"
source_cr: "CR-102"
gate: "CR102-FUTURE-CONCRETE-EXECUTION-AUTHORIZATION"
release_id: "cr102-test-strategy-package-v0.1-20260621T003214Z"
object_label: "test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z"
nas_label: "quant-lab-primary"
operator_label: "current-shell-user"
credential_source: "none-needed"
execution_machine_pull: "PASS-manual-per-file-sha256-bytes"
execution_machine_pull_evidence: "runs/RUN-EXEC-20260621-003.md"
raw_path_redacted: true
credential_read_performed: false
runtime_started: false
trading_performed: false
provider_lake_catalog_publish_performed: false
delete_performed: false
mount_performed: false
network_probe_performed: false
---

# CR102 NAS Real Package Exchange Evidence

## Summary

| Field | Value |
|---|---|
| status | PASS |
| operation | source-create -> package-exchange-copy -> release-publish-label |
| nas_label | quant-lab-primary |
| object_label | test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z |
| operator_label | current-shell-user |
| credential_source | none-needed |
| execution_machine_pull | manual-execution-machine-verification |
| execution_machine_pull_result | PASS |
| execution_machine_pull_evidence | runs/RUN-EXEC-20260621-003.md |
| rollback_status | not-needed |

## Redacted Hash / Count Evidence

| Scope Label | File Count | Bytes | Content Hash |
|---|---:|---:|---|
| staging_source_scope | 3 | 817 | 1aa2c18c7a3d5b9396f4f4adc9592b17c5b553482424d54c144fe0763a46c647 |
| package_exchange_target_scope | 3 | 817 | 1aa2c18c7a3d5b9396f4f4adc9592b17c5b553482424d54c144fe0763a46c647 |
| release_publish_scope | 3 | 817 | 1aa2c18c7a3d5b9396f4f4adc9592b17c5b553482424d54c144fe0763a46c647 |

## Checks

| Check | Result |
|---|---|
| source_temp_hash_before_rename | 1aa2c18c7a3d5b9396f4f4adc9592b17c5b553482424d54c144fe0763a46c647 |
| source_temp_file_count_before_rename | 3 |
| source_temp_bytes_before_rename | 817 |
| hash_match_across_scopes | true |
| count_match_across_scopes | true |
| overwrite_existing_target | false |
| delete_used | false |
| mount_used | false |
| network_probe_used | false |
| credential_env_account_read | false |
| qmt_miniqmt_xtquant_gateway_started | false |
| provider_lake_catalog_publish | false |

## Manual Follow-Up

Execution-machine pull/readback was completed by the execution-host operator as manual verification. Agent did not access execution-machine mount path.

## Execution-Machine Pull / Readback Evidence

| Field | Value |
|---|---|
| status | PASS |
| verification_mode | per-file-sha256-bytes |
| object_label | test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z |
| file_count | 3 |
| mismatch_count | 0 |
| checked_by | execution-host-operator |
| checked_at | 2026-06-21T12:52:05+08:00 |
| run_exec | runs/RUN-EXEC-20260621-003.md |

| Relative Path | Bytes | SHA256 | Result |
|---|---:|---|---|
| README.txt | 112 | 3d52397289472719a654a852096fffea02fbe16087e4b327080cf481cab36036 | PASS |
| manifest.json | 508 | c9a7ad10215b5ffba424c89db517c7f8599a92ed9f3e5cf9cd8dc05f52092626 | PASS |
| payload/strategy_stub.py | 197 | 624355cef61e19f9ec61fffa870260568564a63d94867e0346ca857c64e9b47d | PASS |

`RUN-EXEC-20260621-002` aggregate `content_hash` mismatch is treated as an aggregate-hash canonicalization false negative because per-file relative path / sha256 / bytes all match.
