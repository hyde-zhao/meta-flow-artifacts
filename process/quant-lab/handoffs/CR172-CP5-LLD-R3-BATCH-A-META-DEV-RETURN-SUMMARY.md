---
handoff_id: "CR172-CP5-LLD-R3-BATCH-A"
change_id: "CR-172"
checkpoint: "CP5"
canonical_role: "meta-dev"
story_ids:
  - "CR172-S01-action-authorization-eligibility-governance"
  - "CR172-S02-trial-return-artifact-native-hook"
status: "complete"
completed_at: "2026-07-18T12:58:00+08:00"
---

# CR-172 CP5 LLD R3 Batch A — Return Summary

## 结论

S01/S02 两份 full LLD 已升级为 v1.2，保持 `ready-for-review`、`confirmed=false`、`open_items=0`。R2 findings 在本 batch 的整改义务已闭合：current-v1 的 caller 自报 `approved_ledger` 不能解锁任何 action；REQ-013 诚实标记为 contract ready / runtime enforcement deferred；S04 可以复用 S02 唯一 verifier library，但不能绕过 S03 selected replica 取得数据。normative seal contract 与 R2 完全一致。

## 修改文件

1. `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md`
2. `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md`
3. `process/handoffs/CR172-CP5-LLD-R3-BATCH-A-META-DEV-RETURN-SUMMARY.md`

未修改 HLD/ADR/Feature/Story/Plan、源码、测试、fixture、state、ledger 或 checkpoint。

## S01 R3 整改

| 项 | 冻结合同 | 量化守卫 |
|---|---|---:|
| approved-ledger trust | `evaluate_action_decision` 在识别 `decision_origin=approved_ledger` 后不读取 record allow/path、不检查 predecessor，直接返回 `authorized=false`、`eligible_to_execute=false`、唯一 reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE` | six actions accepted/eligible=`0/0`；consumer callback=`0` |
| 12-field record | `ActionAuthorizationRecordV1` 仍为 exact `12/12`；origin/target不写回 record | extra fields=`0` |
| provenance single truth | 只使用 `ActionDecisionV1.decision_origin + ActionScopeContextV1.target_kind + logical URI/port binding`；同义 provenance field/helper/assertion=`0/0/0` | second truth=`0` |
| REQ-013 | `RunPathDecisionV1` 只交付 value contract；本 CR=`contract_ready/runtime_enforcement_deferred` | runner diff/default switch/runtime enforcement/runtime delivered=`0/0/0/0` |
| future route | 可信 issuer/envelope/adapter 与 native-producer path enforcement 都必须另走 runtime-high-risk CR | current trusted adapter symbols=`0` |

future native producer 必须在 launch/workspace first side effect 前消费 `RunPathDecisionV1`，证明 new default=`1`、legacy write=`0`；在此之前 CP8 不得声称 REQ-013 runtime delivered。

## S02 R3 整改

- authority pointers 已更新为 HLD/ADR v1.3、Feature I01/I03 v1.2。
- R2 normative seal types/functions、digest format和 hash domain均不改变：
  - `SealedTrialReturnBundleV1`
  - `ResearchCanonicalSelectionV1`
  - `VerifiedTrialReturnBundleV1`
  - `canonical_artifact_seal_bytes`
  - `canonical_artifact_seal_sha256`
  - `verify_sealed_trial_return_bundle`
- S04 可以将 3 public types + verifier 作为 library dependency，用于 execution-staging bytes-level `4/4` 复验。
- S04 的 bundle/selection 数据仍只能来自 S03 selected-replica staging port；S04 bypass S03、receipt-only trust、secondary canonicalizer/digest/re-seal=`0/0/0/0/0`。
- S04 verifier-library dependency 不构成 data-source dependency，也不允许从 S02/research canonical/NAS未选择对象旁路取数。
- 上述精确边界已通知 Batch B meta-dev，用于 S03/S04 R3 correlation。

## 授权与实施边界

- 两份 LLD 仍只设计 repository-local contract/fixture 范围。
- source/test/fixture implementation=`0/0/0`；真实 lake/NAS/runtime/lineage/R/signal/trading/migration/deploy/Git remote operation=`0`。
- 六类真实动作 authorized/executed=`0/6`,`0/6`。
- 本轮不授权 CP6，不改变 CP5 pending/review 状态。

## 验证结果

```text
meta-flow story lld-check --lld S01 --evidence-type full-lld
  -> LLD Structure Check: OK

meta-flow story lld-check --lld S02 --evidence-type full-lld
  -> LLD Structure Check: OK

git diff --check -- S01-LLD S02-LLD R3-return-summary
  -> PASS
```

两份 LLD sections=`0～14` 完整；R3 revision entries=`2/2`；clarification active items=`0`。
