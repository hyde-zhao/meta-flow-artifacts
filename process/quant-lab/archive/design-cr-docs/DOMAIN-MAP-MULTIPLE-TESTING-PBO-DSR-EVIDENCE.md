---
status: draft-for-cp3
version: "0.1"
cr_id: "CR-164"
---

# Domain Map: CR164 Computable Statistical Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host-orchestrator inline meta-se-critical | 定义输入、方法证据、聚合状态、raw/effective count 边界与业务规则。 |

## 术语表

| Term | 定义 | 来源 / 限制 |
|---|---|---|
| Sealed Statistical Input | 与 CR163 family ref/hash/raw count 和完整 membership 绑定的规范化输入 | REQ-CR164-002 |
| Method Evidence | 单一方法的输入、配置、结果、provenance、availability 和 reasons | REQ-CR164-003 |
| Raw Trial Count | sealed family 中 distinct stable trial identities 数量 | 可供 DSR raw mode；不是 effective count |
| Effective Trial Count | 考虑相关性的有效试验数 | CR164 不计算，保持 typed_unavailable |
| Claim-relevant Mandatory Set | 某个 claim 必须满足的方法集合 | 不允许 OR-pass；由 schema/version 显式列出 |

## 领域对象

| Object | Owner | 关键属性 | 状态 | 规则来源 |
|---|---|---|---|---|
| StatisticalEvidenceInput | FEAT-164-01 | schema, family_ref/hash, raw_count, candidate_ids, p-values, returns, moments, split inputs | unvalidated/validated/unavailable/blocked | REQ-002/004 |
| MethodConfig | calculator | method, alpha, bootstrap mode/window, seed, split policy, DSR input method | validated/blocked | REQ-003/005 |
| MethodEvidence | calculator | method, availability, result, refs, input/config hash, reasons | pass/fail/typed_unavailable/blocked | REQ-003/006 |
| StatisticalEvidenceSummary | FEAT-164-04 | mandatory set, method refs, aggregate status/reasons, summary hash | pass/fail/typed_unavailable/blocked | REQ-006/007 |
| ConsumerProjection | FEAT-164-04 | consumer id, summary ref/hash, limitations | attached/blocked | REQ-007/009 |

## 状态机与决策表

| 输入条件 | 方法状态 | 聚合影响 |
|---|---|---|
| required input absent but no contradictory fact | `typed_unavailable` | mandatory claim 不得 PASS |
| identity/hash/count mismatch、tamper、NaN/Inf、invalid domain | `blocked` | aggregate `blocked` |
| valid computation does not meet method policy | `fail` | aggregate at least `fail` |
| valid computation meets method policy | `pass` | 仅当所有 claim-relevant mandatory methods PASS 才可 aggregate PASS |

优先级固定为 `blocked > fail > typed_unavailable > pass`。这里的 `>` 表示更保守、更高阻断级别；不表示统计显著性大小。

## 业务规则

| Rule | 规则 | 验证入口 |
|---|---|---|
| RULE-164-01 | family/ref/hash/raw count/membership 绑定覆盖率 100%，count difference=0 | QAC-002/003 |
| RULE-164-02 | BH 与 WRC/SPA 至少 2 个完整候选 | minima fixture |
| RULE-164-03 | PBO 至少 4 候选、4 valid splits 且 train/test 非空 | minima fixture |
| RULE-164-04 | DSR 至少 2 trials、sample_length≥30、finite moments、variance>0 | minima fixture |
| RULE-164-05 | `dsr_input_method` 必须为 `raw_trial_count`；effective fields 空且 unavailable | non-alias fixture |
| RULE-164-06 | WRC/SPA MVP 使用 stationary bootstrap `fixed_window`，window≥1 且 seed/config 入 hash | deterministic fixture |
| RULE-164-07 | 相同 fixture 10 次只产生 1 个 summary hash | QAC-005 |
| RULE-164-08 | UC-58/59/60 projection 3/3；59/60 不表示 adapter implementation | QAC-007 |
| RULE-164-09 | CR155 1/1 blocked；所有 forbidden counters=0；overclaim=0 | QAC-008/009/010 |

