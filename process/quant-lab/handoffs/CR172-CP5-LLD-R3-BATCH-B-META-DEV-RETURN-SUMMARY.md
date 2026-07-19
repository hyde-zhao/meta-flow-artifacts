---
handoff_id: "CR172-CP5-LLD-R3-BATCH-B"
change_id: "CR-172"
checkpoint: "CP5"
canonical_role: "meta-dev"
story_ids: ["CR172-S03-nas-replica-verification", "CR172-S04-execution-cache-materialization"]
status: "complete"
completed_at: "2026-07-18T12:30:55+08:00"
---

# CR-172 CP5 LLD R3 Batch B Return Summary

## 结论

S03/S04 两份 LLD 已完成 R3 最小、不扩权回修。两份 LLD 均为 `lld_version=1.2`、`status=ready-for-review`、`confirmed=false`、`open_items=0`；R2 finding F-CR172-CP5-R2-002 的 S04 bytes-level seal verification 缺口已在本 batch 边界内关闭，可以进入 Host cross-contract correlation 与独立 QA R3。

## 修改文件（3）

1. `process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md`
2. `process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md`
3. `process/handoffs/CR172-CP5-LLD-R3-BATCH-B-META-DEV-RETURN-SUMMARY.md`

其他 LLD、Story 卡片、HLD/ADR/Feature、源码、测试、fixture、STATE、ledger、checkpoint 修改=`0`。

## R3 闭环结果

| 项 | 冻结结果 | 量化守卫 |
|---|---|---:|
| S03 selected-replica source | `ReplicaStoragePort.read_selected_replica(current DistributionSelectionV1)` 精确返回 immutable `SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1 + ReplicaVerificationReceiptV1`；selection 的 replica-version/receipt/source-selection refs 必须全部一致 | output types/ref match=`3/3`；unselected/staging/path/latest/receipt-only accepted=`0/5` |
| S03 边界 | read contract 只负责 S03 selection-bound 数据选择，不验证 seal、不 canonicalize、不 hash；S04 继续直接消费 S02 verifier-library | 新增 digest/verifier-facade/data-bypass=`0/0/0` |
| S04 三依赖分离 | S01 只提供 decision/context；S03 current selection 是唯一数据来源；S02 只提供 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1` 和 `verify_sealed_trial_return_bundle` | S03 data-source call=`1`；S02 data-source call=`0`；verifier truth=`1` |
| typed staging | `MaterializationStoragePort.pull_to_staging(...)` 必须委托 S03 selected read，返回 `ExecutionStagingTokenV1 + SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1` | typed output=`3/3`；bypass-S03/receipt-only/S02-data lookup=`0/0/0` |
| bytes-level 4/4 | S04 对 staging bundle+selection 调 S02 verifier exactly once；成功后才以 verified result、expected 与 S03 receipt计算 release/manifest/seal/content `4/4` | verifier call=`1` per complete candidate；receipt-only-trust/secondary-seal-digest=`0/0` |
| tampered seal negative | `T-S04-F03` 对 selected staging bundle 的 seal bytes 做 mutation | tampered seal bytes→seal=`false`/pointer=`0`；cache/receipt=`0/0` |
| 测试 ID | 沿用 Feature v1.2 冻结集合，不新增第 17 个 ID | total/unique/duplicate=`16/16/0` |

## 验证结果

- S03 `meta-flow story lld-check --evidence-type full-lld`：PASS（`LLD Structure Check: OK`）。
- S04 `meta-flow story lld-check --evidence-type full-lld`：PASS（`LLD Structure Check: OK`）。
- frontmatter：v1.2/ready-for-review/confirmed=false/open_items=0=`2/2` PASS。
- S04 test IDs：`total=16 unique=16 duplicate=0`。
- selected-replica exact tuple、typed staging tuple、S02 verifier、tampered-seal、zero-bypass 静态合同检查：PASS。
- targeted `git diff --check -- <S03-LLD> <S04-LLD>`：PASS。

## 边界

本 batch 只修订设计证据。真实 lake/NAS/执行机读写、sync/pull/materialize、multi-trial runtime、trial-return 生成、empirical R、signal、trading、deploy、migration 与 Git remote 操作均为 `0`；CP5 全量人工确认前不得进入实现。
