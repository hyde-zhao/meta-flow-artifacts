---
handoff_id: "CR172-CP5-LLD-R2-BATCH-A"
change_id: "CR-172"
checkpoint: "CP5"
canonical_role: "meta-dev"
story_ids: ["CR172-S01-action-authorization-eligibility-governance", "CR172-S02-trial-return-artifact-native-hook"]
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
architecture_return_ref: "process/handoffs/CR172-CP5-LLD-R2-ARCHITECTURE-CORRECTION-META-SE-CRITICAL-RETURN-SUMMARY.md"
status: "ready"
---

# CR-172 CP5 LLD R2 Batch A

只修改 S01/S02 两份现有 LLD，并写本 batch return summary；不得修改 HLD/ADR/Feature/Story/Plan、源码、测试、fixture、state、ledger、checkpoint。

必须关闭 R1 F-001～004：

- S01：approval record 仍为 12 fields；冻结 `ActionDecisionOriginV1(repository_fixture|approved_ledger)`、`ActionScopeContextV1.target_kind(repository_fixture|real_operation)`，fixture+real 在 first side effect 前接受数=0。
- S02：彻底删除 mature runner hook、turnover/net_forward_return mapping、lineage finish/pointer rollback 和成功 lineage 痕迹=0 的旧承诺；primary paths 仅 `engine/trial_return_artifact.py` 与本 Story test。
- S02 变为 pure schema + repository fixture producer port；`forward_label_proxy@v1` 进入 trial-return/empirical-R/effective-count 接受=`0/0/0`。
- S02 导出精确 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`canonical_artifact_seal_bytes`、`canonical_artifact_seal_sha256`、`verify_sealed_trial_return_bundle`；digest 格式为 `sha256:` + lowercase 64 hex，hash domain 只有同一 canonical seal bytes。
- lineage partial success 只作为 future integration 风险：状态=`partial_lineage_blocked_audit`，erase/fake rollback/canonical advance=`0/0/0`；原子能力转独立 lineage-owner CR。
- 两份 LLD 修订记录版本升为 1.1，frontmatter 保持 `ready-for-review`、`confirmed=false`、`open_items=0`；通过 `meta-flow story lld-check`。

return summary：`process/handoffs/CR172-CP5-LLD-R2-BATCH-A-META-DEV-RETURN-SUMMARY.md`。
