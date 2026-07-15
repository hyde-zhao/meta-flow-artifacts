---
status: approved-cp3
version: "0.3"
change: CR-170
source_hld: process/archive/design-cr-docs/HLD-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md
---

# Canonical Reliability N/A Semantics and Admission Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-15 | host-orchestrator inline meta-se-critical | 定义 21-unit policy、五态 decision、结构化 N/A boundary 与 tier admission 领域规则。 |
| 0.2 | 2026-07-15 | host-orchestrator inline meta-se-critical | 增加 baseline N/A path、hardening direction、complete-N/A disposition、boundary authoring contract；冻结 15/5/1 分组、G1-P06 prohibited 与 T3 existing-behavior invariant。 |
| 0.3 | 2026-07-15 | host-orchestrator inline | 回填 CP3 已批准状态并迁入设计归档；不改变领域对象或不变量。 |

## 领域对象

| Object | Owner | 关键字段 | 不变量 |
|---|---|---|---|
| `NaPolicySpec` | FEAT-14 | policy_id、gate_id、evidence_family、applicability、owner、boundary_rule、result_rule、baseline_na_path_type、hardening_direction、complete_na_disposition | 固定 21/21、ID 唯一、字段完整 100%；方向分组 15/5/1 |
| `NaBoundary` | FEAT-14 | reason、owner、scope、release_profile/authorization_ref | complete=4/4 且 owner/scope 匹配 |
| `NaBoundaryAuthoringContract` | FEAT-14 | current_writers、future_writers、authorization_ref_semantics、synthesis_allowed | current=fixture/test；future=FU-009/Stage3 CR；synthesis=false |
| `NaEvidenceDecision` | FEAT-14 | policy_id、applicable、state、reason_category、boundary、claim_id | state 仅五种；deterministic |
| `ReliabilityGateSummary` | FEAT-14 | gate_id、status、refs、blocked_claims、reason | 公共 schema 不变；negative mandatory state 非 PASS |
| `AdmissionPolicyResult` | FEAT-14/07 | tier、gate_mode、status、wording、claims | T0 NR、T1/T2 BLOCKED、T3 mode NOT_AUTHORIZED |
| `VerifierConsumerBoundary` | FEAT-08 | current_consumer、future_consumer、availability、risk_ref | current=Gate maintainer；future=FU-006 verifier |

## 状态机

```text
raw evidence
  -> PRESENT
  -> MISSING
  -> NA_WITH_COMPLETE_BOUNDARY
  -> NA_WITH_INCOMPLETE_BOUNDARY
  -> GENERIC_REASON_ESCAPE

applicable mandatory COMPLETE_NA + reviewable -> NEEDS_REVIEW
applicable mandatory COMPLETE_NA + prohibited -> BLOCKED
MISSING/INCOMPLETE/GENERIC -> non-PASS claim
Gate summaries -> protected worst-state -> tier admission
```

## 核心规则

| Rule | 规则 |
|---|---|
| RULE-CR170-01 | 通用 `na_reason/n_a_reason` 满足 mandatory policy unit 的数量必须为 0。 |
| RULE-CR170-02 | complete N/A 必须具备 reason/owner/scope/profile-or-authorization 4/4，且 applicable mandatory unit 不得 PASS。 |
| RULE-CR170-03 | conditional unit 不适用时仍保留 inventory 行并记录 applicability；不得缩小 21 分母。 |
| RULE-CR170-04 | bottom-up NEEDS_REVIEW 必须保留；无失败证据时 merge production 修改=0。 |
| RULE-CR170-05 | T0/T1/T2/T3 对 mandatory NEEDS_REVIEW 的结果固定为 NR/BLOCKED/BLOCKED/NOT_AUTHORIZED semantics。 |
| RULE-CR170-06 | future verifier 不等于 available verifier；FU-006 完成前不声明 independent verification。 |
| RULE-CR170-07 | 21-unit 现有路径与方向固定为更严/受控放宽/保持 15/5/1；放宽组只能到 NEEDS_REVIEW，T1/T2 仍 BLOCKED。 |
| RULE-CR170-08 | G1-P06 的 complete_na_disposition=prohibited；trial count/value/provenance 不得被 N/A 替代。 |
| RULE-CR170-09 | `n_a_boundaries`/`authorization_ref` 由 caller 显式提供；evaluator 不合成、不读取凭据、不提升权限。 |
| RULE-CR170-10 | T3 现有 early-return 保持 status=BLOCKED + mode=NOT_AUTHORIZED；兼容回归 1/1，生产修改=0。 |

## 术语

| Term | 定义 |
|---|---|
| canonical | 项目内被多个 caller 共同消费、决定统一 Gate 语义的权威实现，不等于真实数据或生产授权。 |
| generic reason escape | 未绑定具体 policy/owner/scope 的通用 reason 被用来替代一个或多个 mandatory evidence。 |
| complete boundary | reason、owner、scope、profile 或 authorization ref 完整并与 policy 匹配。 |
| controlled widening | 仅把适用 unit 的 Gate-local 历史 BLOCKED 改为带 claim 的 NEEDS_REVIEW；不得产生 PASS，且 T1/T2 admission 仍 BLOCKED。 |
| authorization ref | caller 显式提供的非敏感审计指针；不是 credential，也不代表本轮获得新授权。 |
| worst-state | 多个状态聚合时保留最保守状态，不把 NEEDS_REVIEW/BLOCKED/FAIL 升级为 PASS。 |
