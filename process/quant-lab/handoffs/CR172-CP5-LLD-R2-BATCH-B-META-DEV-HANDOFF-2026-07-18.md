---
handoff_id: "CR172-CP5-LLD-R2-BATCH-B"
change_id: "CR-172"
checkpoint: "CP5"
canonical_role: "meta-dev"
story_ids: ["CR172-S03-nas-replica-verification", "CR172-S04-execution-cache-materialization"]
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
architecture_return_ref: "process/handoffs/CR172-CP5-LLD-R2-ARCHITECTURE-CORRECTION-META-SE-CRITICAL-RETURN-SUMMARY.md"
status: "ready"
---

# CR-172 CP5 LLD R2 Batch B

只修改 S03/S04 两份现有 LLD，并写本 batch return summary；不得修改共享基线、源码、测试、fixture、state、ledger、checkpoint。

必须关闭 R1 F-003～005：

- S03 精确消费 S01/S02 public contract：`ActionDecisionV1`、`ActionScopeContextV1`、`SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`verify_sealed_trial_return_bundle`。
- 删除不存在的 `AuthorizationContextV1`；fixture origin 只能绑定 repository_fixture target。
- `ReplicaVerificationReceiptV1.original_seal_sha256` 必须直接取 verifier 返回的 `VerifiedTrialReturnBundleV1.original_seal_sha256`；S03 secondary digest/canonicalizer=`0/0`。
- S04 精确消费 S03 receipt/selection，继续强制 fixture origin/target；测试表必须 16 个唯一 ID、duplicate=0，修掉重复 `T-S04-N03`。
- 两份 LLD 版本升为 1.1，保持 ready-for-review/confirmed=false/open_items=0；通过 lld-check。

return summary：`process/handoffs/CR172-CP5-LLD-R2-BATCH-B-META-DEV-RETURN-SUMMARY.md`。
