---
handoff_id: "CR172-CP5-LLD-R3-MINIMAL-CORRECTION"
change_id: "CR-172"
checkpoint: "CP5"
role: "meta-se-critical"
status: "complete"
completed_at: "2026-07-18T12:35:00+08:00"
review_source: "process/checks/CP5-CR172-LLD-REVIEW-R2-FINDINGS.md"
---

# CR-172 CP5 LLD R3 最小架构整改 Return Summary

## 结论

R2 findings 的四项最小、不扩权整改已落盘。没有新增真实授权、实现范围或 runtime claim；五份 LLD、源码、测试、fixture、state、ledger、checkpoint 均未修改。五张 Story 已转 `lld-rework-required-r3`，CP5 batch 已转 `lld-r3-required`。

## 修改文件（16）

1. `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`（v1.3）
2. `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`（v1.3）
3. `docs/design/FEATURE-DESIGN-MATRIX.md`（v1.30）
4. `docs/features/trial-return-artifact-pipeline/DESIGN.md`（v1.2）
5. `docs/features/research-artifact-replica-materialization/DESIGN.md`（v1.2）
6. `docs/features/research-artifact-replica-materialization/TEST-PLAN.md`（v1.2）
7. `docs/features/research-artifact-replica-materialization/TASKS.md`（v1.1）
8. `docs/features/path-i-authorization-claim-governance/DESIGN.md`（v1.2）
9. `docs/features/path-i-authorization-claim-governance/TEST-PLAN.md`（v1.2）
10. `docs/features/path-i-authorization-claim-governance/TASKS.md`（v1.1）
11. `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance.md`
12. `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook.md`
13. `process/stories/STORY-CR172-S03-nas-replica-verification.md`
14. `process/stories/STORY-CR172-S04-execution-cache-materialization.md`
15. `process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification.md`
16. `process/DEVELOPMENT-PLAN.yaml`

## 四项闭环

| Finding | 冻结结果 | 量化结果 |
|---|---|---:|
| approved-ledger trust | current-v1 所有 `approved_ledger` 输入固定 `authorized=false`、`eligible_to_execute=false`，reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；caller 自报 enum/record 不能解锁 | accepted/eligible=`0/0` |
| S04 bytes-level seal verify | 数据只能来自 S03 selected replica；staging port 返回 sealed bundle+selection；S04 直接依赖 S02 唯一 verifier library；receipt 不替代 bytes | bypass/receipt-only/secondary-digest=`0/0/0` |
| fixture provenance | 唯一来源为 `ActionDecisionV1.decision_origin + ActionScopeContextV1.target_kind + fixture URI/port` | `evidence_kind` field/helper/assertion=`0` |
| REQ-013 | 本 CR=`contract_ready/runtime_enforcement_deferred`；future native-producer path-enforcement 在 launch/workspace first side effect 前消费 `RunPathDecisionV1` | current runtime enforcement/default switch/runtime-delivered=`0/0/0` |

## LLD R3 精确输入

1. **S01**
   - 保持 12-field record 不变。
   - current-v1 `approved_ledger` 无条件双 false并返回稳定 reason。
   - 删除所有 `evidence_kind` 定义、别名和测试辅助字段。
   - `RunPathDecisionV1` 仅 contract-ready；不得产生 runtime default-switch claim。
2. **S02**
   - 沿用 R2 pure artifact/verifier contract。
   - 明示 future native-producer 前置包含 path-enforcement：launch/workspace first side effect 前消费 `RunPathDecisionV1`，new default=1、legacy write=0。
3. **S03**
   - 向 S04 暴露 selected-replica staging port，精确返回 `SealedTrialReturnBundleV1 + ResearchCanonicalSelectionV1`。
   - receipt/selection 仍是唯一数据来源授权边界；不得改变 S02 verifier 真相。
4. **S04**
   - 增加 S02 `verifier-library` contract dependency，同时保留 S03 selected-replica runtime-selection dependency。
   - staging bundle 必须调用 S02 唯一 verifier 做 bytes-level `4/4` 复验。
   - tampered seal bytes→seal=false、cache pointer advance=0；不得 receipt-only trust或第二 digest。
5. **S05**
   - 删除 `evidence_kind=fixture` 第二真相。
   - 断言 approved-ledger accepted/eligible=0/0。
   - 断言 S04 bypass/receipt-only/secondary-digest=0/0/0。
   - 断言 REQ-013 runtime path enforcement/default switch/runtime-delivered claim=0/0/0。

## Development Plan 变化

- S04 新增 `S02 -> S04 verifier-library` contract edge；数据路径仍为 `S03 -> S04 runtime-selection`。
- CR172 DAG 从 `5 nodes/9 edges` 调整为 `5 nodes/10 edges`，串行 merge order 不变：`S01→S02→S03→S04→S05`。
- S04 Feature refs 扩为 I01/I02/I03；file owner 和 primary paths 不变。

## OPEN / Deferred

| 项 | 状态 | 边界 |
|---|---|---|
| trusted approved-ledger issuer/envelope/adapter | OPEN-deferred | 独立 runtime-high-risk CR；当前 v1 永远双 false |
| native-producer runtime path enforcement | OPEN-existing-sub-prerequisite | 绑定既有 future native-producer CR；当前 runner diff=0 |
| empirical v2 / public C1 projection、atomic lineage | OPEN-existing | 沿既有独立 owner CR 路线，不进入本次整改 |

新增 Spike=`0`；新增真实授权=`0`。

## Host 下一步

按更新后的 10-edge DAG 生成五份 LLD R3，完成 cross-contract correlation 后重新发起独立 QA review。R3 review 通过前不得启动 CP5 人工批准或实现。
