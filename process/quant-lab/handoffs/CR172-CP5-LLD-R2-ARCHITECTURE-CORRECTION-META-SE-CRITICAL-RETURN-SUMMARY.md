---
handoff_id: "CR172-CP5-LLD-R2-ARCHITECTURE-CORRECTION"
change_id: "CR-172"
checkpoint: "CP5"
role: "meta-se-critical"
status: "complete"
completed_at: "2026-07-18T11:38:48+08:00"
review_source: "process/checks/CP5-CR172-LLD-REVIEW-R1-FINDINGS.md"
---

# CR-172 CP5 LLD R2 架构整改 Return Summary

## 结论

F-CR172-CP5-001～005 的上游架构基线已按 Host 指定的安全收窄路线完成整改，可以交给 meta-dev 进行五份 LLD R2。当前 CR 不再修改现有 mature runner，也不再把 `net_forward_return` 映射为 trial portfolio return；append-only lineage partial success 被诚实固定为 BLOCKED audit lane。

## 修改文件（16）

1. `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`（v1.2）
2. `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`（v1.2）
3. `docs/design/FEATURE-DESIGN-MATRIX.md`（v1.29）
4. `docs/features/trial-return-artifact-pipeline/DESIGN.md`（v1.1）
5. `docs/features/trial-return-artifact-pipeline/TEST-PLAN.md`（v1.1）
6. `docs/features/trial-return-artifact-pipeline/TASKS.md`（v1.1）
7. `docs/features/research-artifact-replica-materialization/DESIGN.md`（v1.1）
8. `docs/features/research-artifact-replica-materialization/TEST-PLAN.md`（v1.1）
9. `docs/features/path-i-authorization-claim-governance/DESIGN.md`（v1.1）
10. `docs/features/path-i-authorization-claim-governance/TEST-PLAN.md`（v1.1）
11. `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance.md`
12. `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook.md`
13. `process/stories/STORY-CR172-S03-nas-replica-verification.md`
14. `process/stories/STORY-CR172-S04-execution-cache-materialization.md`
15. `process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification.md`
16. `process/DEVELOPMENT-PLAN.yaml`

五份 LLD、源码、测试、fixture、STATE、ledger、checkpoint 均未修改。Story 状态与 CP5 batch 已改为 `lld-rework-required-r2` / `lld-r2-required`。

## 五项整改结果

| Finding | 整改结果 | 量化守卫 |
|---|---|---:|
| F-001 lineage | 不扩 lineage owner；partial success 不擦除、不用 `fail()` 伪装回滚，输出 `partial_lineage_blocked_audit`；真实 canonical selection 不推进 | erase/fake rollback/advance=`0/0/0` |
| F-002 return semantics | 现有 `net_forward_return` 定类为 `forward_label_proxy@v1`；本 CR pure contract + fixture port，现有 runner/lineage diff=0 | proxy→trial-return/R/count=`0/0/0`；runner/lineage diff=`0/0` |
| F-003 fixture/real | 保持 approval record 12 fields；派生 decision/context 新增 `decision_origin`/`target_kind` | enums=`2/2`；fixture+real accepted=`0` |
| F-004 seal | 冻结 `canonical_artifact_seal_bytes`、`canonical_artifact_seal_sha256`、`verify_sealed_trial_return_bundle` 和 `VerifiedTrialReturnBundleV1.original_seal_sha256` | seal truth=`1/1/1`；S03 secondary digest=`0` |
| F-005 S04 IDs | 上游 TEST-PLAN 固定 16 个唯一 ID | unique/duplicate=`16/0` |

## LLD R2 精确修改要求

1. **S01**：12-field record 不变；`ActionDecisionV1.decision_origin=repository_fixture|approved_ledger`；`ActionScopeContextV1.target_kind=repository_fixture|real_operation`；fixture+real 在 first side effect 前 deny。
2. **S02**：删除 mature runner、既有 lineage test 和 `turnover` mapping；primary 仅 `engine/trial_return_artifact.py` + 本 Story test。实现 pure schema/fixture port、forward-proxy deny、唯一 seal digest/verifier、`partial_lineage_blocked_audit`；导出 3 个精确类型和 verifier 合同。
3. **S03**：精确消费 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`VerifiedTrialReturnBundleV1`、`ActionDecisionV1`、`ActionScopeContextV1` 与 `verify_sealed_trial_return_bundle`；receipt 的 `original_seal_sha256` 只能来自 verified result。
4. **S04**：消费 S03 receipt 并保留 fixture origin/target；16 个测试 ID 必须逐一唯一，duplicate=0；真实 NAS/execution target 始终 deny。
5. **S05**：跨合同断言 runner/lineage diff=`0/0`、forward proxy accepted=0、fixture+real accepted=0、partial lineage selection=0、S03 secondary digest=0、六类真实动作 authorized/executed=`0/6`,`0/6`。

## OPEN / Deferred

| 项 | 状态 | Owner / 启动条件 |
|---|---|---|
| 真实 native multi-trial period-return producer/instrumentation CR | OPEN-deferred-prerequisite | runtime/data owner；必须提供区间端点、持仓/权重、成本、return basis、non-overlap/alignment，走 runtime-high-risk |
| append-only lineage 原子批次/outbox/correction-supersession CR | OPEN-deferred-prerequisite | lineage owner；不得由 CR-172 扩权代做 |
| empirical method v2 / public C1 projection | OPEN-existing | methodology/C1 owners；保持现有 FU/独立 CR 路线 |

当前 CR 无需新增 Spike；上述任一前置不足时真实 source 维持 `typed_unavailable`，partial lineage 维持 `BLOCKED`。

## 校验

- `git diff --check`：PASS。
- `process/DEVELOPMENT-PLAN.yaml`：PyYAML parse PASS（`YAML_OK`）。
- 旧 positive runner hook、`net_forward_return`→trial-return mapping、lineage success trace=`0` 承诺的 active-scope scan：0 条；只保留明确的 deny/zero-diff/forward-proxy 分类与不可变 Story ID。
- S04 test ID：16 个唯一 token、重复 0。
- 实际源码/测试/fixture/真实操作：`0/0/0/0`；六类真实动作 authorized/executed=`0/6`,`0/6`。

## Host 下一步

按 S01→S02→S03→S04→S05 顺序派发 LLD R2；五份 R2 通过结构检查和 cross-contract correlation 后，重新发起独立 QA review。R2 review 通过前不得启动 CP5 人工批准，也不得进入实现。
