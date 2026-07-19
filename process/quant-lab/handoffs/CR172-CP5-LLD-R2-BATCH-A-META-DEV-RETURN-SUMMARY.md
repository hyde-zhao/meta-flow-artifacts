---
handoff_id: "CR172-CP5-LLD-R2-BATCH-A"
change_id: "CR-172"
checkpoint: "CP5"
canonical_role: "meta-dev"
story_ids:
  - "CR172-S01-action-authorization-eligibility-governance"
  - "CR172-S02-trial-return-artifact-native-hook"
status: "complete"
completed_at: "2026-07-18T12:02:00+08:00"
---

# CR-172 CP5 LLD R2 Batch A — Return Summary

## 结论

S01/S02 两份 full LLD 已升级为 v1.1、保持 `ready-for-review` / `confirmed=false` / `open_items=0`，并关闭 R1 F-001～004 在本 batch 的全部整改义务。当前设计不再修改现有 mature runner，不再把 `forward_label_proxy@v1` 重标为 trial-return，也不承诺 append-only lineage 可以回滚；S02 收窄为 pure schema、repository fixture producer port 和唯一 verified-seal handoff。

## 修改文件

1. `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md`
2. `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md`
3. `process/handoffs/CR172-CP5-LLD-R2-BATCH-A-META-DEV-RETURN-SUMMARY.md`

除此之外未修改 HLD/ADR/Feature/Story/Plan、源码、测试、fixture、STATE、ledger 或 checkpoint。

## R1 finding 关闭证据

| Finding | R2 关闭方式 | 量化守卫 |
|---|---|---:|
| F-CR172-CP5-001 append-only lineage 不可原子回滚 | S02 删除 runner lineage finish/fail/selection rollback 流程；只保留纯 `AppendOnlyLineageAuditV1` / `partial_lineage_blocked_audit`，真实原子能力转独立 lineage-owner CR | erase/fake rollback/canonical advance=`0/0/0` |
| F-CR172-CP5-002 forward-label proxy 非真实 period return | S02 删除 turnover mapping 与现有 runner hook；只接受显式 interval period-return fixture，`forward_label_proxy@v1` 在 candidate 前 fail-closed | proxy→trial-return/R/effective-count=`0/0/0`；runner hook/diff=`0/0` |
| F-CR172-CP5-003 fixture/real 无机械绑定 | S01 保持 approval record exact `12/12`，新增 `ActionDecisionOriginV1`、`ActionTargetKindV1`、`ActionScopeContextV1.target_kind`；S02 在 first port side effect 前强制 fixture origin/target/URI/port 四者一致 | enum values=`2/2`；fixture+real accepted/side-effect=`0/0`；record extra fields=`0` |
| F-CR172-CP5-004 S02→S03 seal/input 合同不精确 | S02 冻结 3 个精确 public types、2 个 canonical seal helpers 和唯一 verifier；verified result 持有唯一 `original_seal_sha256`，S03 禁止 second digest/re-seal | public types/functions=`3/3`,`3/3`；seal truth=`1/1/1`；secondary digest=`0` |

## 精确公共合同

### S01 governance

- `ActionDecisionOriginV1 = repository_fixture | approved_ledger`
- `ActionTargetKindV1 = repository_fixture | real_operation`
- `ActionScopeContextV1(schema_version, scope_revision, scope_sha256, release_id, run_id, family_id, target_kind)`
- `ActionDecisionV1` 在原字段上显式携带 `decision_origin`、`target_kind`
- `evaluate_action_decision(request, record, predecessor_evidence=(), *, decision_origin, evaluated_at) -> ActionDecisionV1`
- `require_action_eligible(decision, *, expected_kind, expected_context, expected_origin=None) -> None`

### S02 artifact → S03 replica

- `SealedTrialReturnBundleV1(payload, manifest, manifest_sha256, seal)`
- `ResearchCanonicalSelectionV1(selection_version, release_id, logical_uri, content_sha256, manifest_sha256, original_seal_sha256, decision_origin, target_kind, selected_at)`
- `VerifiedTrialReturnBundleV1(bundle, selection, original_seal_sha256)`
- `canonical_artifact_seal_bytes(seal) -> bytes`
- `canonical_artifact_seal_sha256(seal) -> "sha256:" + lowercase 64 hex`
- `verify_sealed_trial_return_bundle(bundle, selection) -> VerifiedTrialReturnBundleV1`

S03 receipt 的 `original_seal_sha256` 只能直接取 `VerifiedTrialReturnBundleV1.original_seal_sha256`；S03 import/recompute secondary canonicalization/digest/re-seal=`0/0/0`。上述签名已同步告知 Batch B meta-dev。

## 文件与授权边界

- S02 primary paths 精确为 `engine/trial_return_artifact.py` 与 `tests/research/test_cr172_trial_return_artifact.py`。
- 现有 mature runner、lineage modules/stores、既有 lineage tests 均是 forbidden/read-zero-diff 面。
- 本轮实际 source/test/fixture 修改=`0/0/0`；真实 lake/NAS/runtime/lineage/R/signal/trading/migration/deploy/Git remote operation=`0`。
- 六类真实动作 authorized/executed=`0/6`,`0/6`；本 CR 不提供 approved-ledger adapter。

## 校验结果

```text
meta-flow story lld-check --lld S01 ...  -> LLD Structure Check: OK
meta-flow story lld-check --lld S02 ...  -> LLD Structure Check: OK
git diff --check -- S01-LLD S02-LLD     -> PASS
```

两份 LLD sections=`0～14` 完整，revision entries 已新增，clarification active items=`0`。R2 仍只是设计证据，不授权 T02～T04 实现或任何真实 operation。
