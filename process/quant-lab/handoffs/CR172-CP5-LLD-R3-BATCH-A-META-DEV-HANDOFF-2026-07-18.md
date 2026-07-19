# CR-172 CP5 LLD R3 Batch A

只修改 S01/S02 LLD 和 return summary `process/handoffs/CR172-CP5-LLD-R3-BATCH-A-META-DEV-RETURN-SUMMARY.md`。

- S01：current-v1 所有 `approved_ledger` 输入固定双 false，reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；caller 自报 accepted/eligible=0/0；删除全部 `evidence_kind`；REQ-013=`contract_ready/runtime_enforcement_deferred`，runtime delivered=0。
- S02：仅同步 HLD/ADR/Feature v1.3/v1.2 指针，并精确声明 S04 可把 verifier 作为 library dependency 消费，但不得旁路 S03 selection；normative seal contract不改变。
- 升 LLD v1.2，修订记录完整，ready-for-review/confirmed=false/open_items=0；两份 lld-check + targeted diff-check PASS。
- 不得改其他文件、源码、测试、fixture、state/ledger/checkpoint。
