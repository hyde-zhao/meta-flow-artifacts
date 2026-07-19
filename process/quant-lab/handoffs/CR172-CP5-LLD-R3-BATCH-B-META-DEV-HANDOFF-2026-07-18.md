# CR-172 CP5 LLD R3 Batch B

只修改 S03/S04 LLD 和 return summary `process/handoffs/CR172-CP5-LLD-R3-BATCH-B-META-DEV-RETURN-SUMMARY.md`。

- S03：精确声明 selected replica 向 S04 暴露 immutable sealed bundle + `ResearchCanonicalSelectionV1` + receipt；不新增 digest/facade，数据仍由 S03 selection唯一选定。
- S04：依赖 S01 decision/context + S02 verifier-library + S03 selected replica/receipt；`pull_to_staging` 必须返回 typed sealed bundle+selection；对 staging 调用 `verify_sealed_trial_return_bundle` 后再完成 release/manifest/seal/content `4/4`，receipt-only trust/bypass-S03/secondary digest=0/0/0；增加 tampered seal bytes→seal=false/pointer=0。
- 升 LLD v1.2，保留 S04 test IDs 16/16/0，ready-for-review/confirmed=false/open_items=0；两份 lld-check + targeted diff-check PASS。
- 不得改其他文件、源码、测试、fixture、state/ledger/checkpoint。
