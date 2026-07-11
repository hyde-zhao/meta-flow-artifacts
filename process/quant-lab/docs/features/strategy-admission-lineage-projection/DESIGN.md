---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-22"
feature_name: "Strategy-admission lineage projection"
source_hld: "docs/design/HLD-TRIAL-LINEAGE-INSTRUMENTATION.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md#cr163-cp4-增量trial-lineage-instrumentation"
related_stories: ["CR163-S04-existing-admission-projection", "CR163-S05-integrity-recovery-permission-regression"]
lld_policy_summary: "S04/S05 full-lld"
---

# Feature Design: Strategy-admission Lineage Projection

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-11 | meta-se-critical | 冻结 validation projection、三 consumer surfaces、manual reconciliation、claim ceiling 与 status precedence。 |

## 目标与边界

将 core validator 的 immutable projection 接入现有 CR151 statistical gate、CR154 reliability gates 与 `StrategyAdmissionPackage`，不新建 gate、不拥有 lineage、不修改 runtime authorization。`present` 只来自 matching sealed ref/hash + all checks PASS。

## 现有位置与规划

| Path | 当前事实 | 规划变更 |
|---|---|---|
| `engine/strategy_admission_statistical_gate.py` | overfit report消费 caller-supplied `trial_count` | 接受/派生 validated raw count/ref/availability；missing native ref不能 present |
| `engine/cross_strategy_reliability_gates.py` | `raw_trial_count/effective_trial_count/provenance_ref` policy | 适配 raw lineage projection；effective 继续 unavailable，不伪填满足 gate |
| `engine/strategy_admission_package.py` | attach summaries并取更差 status | attach family lineage summary/refs/blocked reason；不改善更差 status，不改 auth flags |

## Projection contract

| 条件 | availability | lineage/raw | effective/C1 | Consumer effect |
|---|---|---|---|---|
| native sealed + validation target match/all PASS | present | ref/hash/raw count | effective unavailable, ref/method empty；C1 non-computable | 只满足 raw lineage input |
| no native instrumentation | typed_unavailable | empty/reason | unavailable | fail closed |
| explicitly excluded path | not_applicable_with_reason | reason/scope | unavailable | 不进入 included denominator |
| invalid/incomplete/tampered/conflict/count mismatch | blocked | machine reasons/evidence ref | unavailable | existing status取更差 |

Legacy manual count：无 sealed ref只作诊断并 typed_unavailable；与 validated raw count一致可标 reconciliation match但不是 truth source；不一致 blocked。不得复制 raw到 effective。

## API / call direction

Producer/core不 import consumers。S04 adapter/consumer调用 `project_family_evidence`，再把 typed payload传入三 existing surfaces。Package attachment必须使用 existing `_worse_admission_status` 等价 precedence。Validation result target ref/hash mismatch视为 tamper blocked。

## 失败、兼容与回退

- Missing fields：typed unavailable或blocked，不使用 default positive count。
- Existing callers不提供 projection：保持原历史路径的 fail-closed语义，不被 silent present。
- CR155：无 native family ledger，保持 blocked、paper_candidate=false；不从现有 artifacts 推断 family/count。
- Rollback：停用 projection adapter，consumer回 unavailable；不删除 core artifacts、不新建 parallel gate。

## LLD 与测试重点

S04 full-lld 冻结三个函数/DTO改动、compatibility signature、status mapping、manual reconciliation表、effective empty representation。S05 full-lld覆盖 positive/unavailable/blocked、manual mismatch、CR155与 forbidden counters。

## Gotchas

- lineage present 不能使 statistical/reliability/package 从 blocked 变 PASS。
- CR154 当前可能要求 effective>=1；CR163 不满足它是正确结果，不得为通过测试伪填。
- `not_applicable_with_reason` 不适用于 CPI-001..004 included mapping。
- package evidence ref不能替代 validator target binding。

