---
feature_id: "FEAT-15"
feature_name: "Cross-Strategy Production Reliability Gates"
change_id: "CR-170"
baseline_change_id: "CR-154"
status: "ready-for-cp5-review"
version: "0.3"
created_at: "2026-07-03T10:40:00+08:00"
owner: "host-orchestrator"
source_hld: "process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
cr170_source_hld: "process/archive/design-cr-docs/HLD-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md"
cr170_source_adr: "process/archive/design-cr-docs/ARCHITECTURE-DECISION-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md"
implementation_allowed: false
authorization_boundary: "local/static/fixture-only; no LLD approval, source implementation, tests implementation, real lake/NAS/provider/QMT/runtime/simulation/paper/live/trading/broker/credential/feed/order/reconciliation/store/catalog/registry/publish"
---

# Cross-Strategy Production Reliability Gates Feature Design

## Purpose

Define the CP4 implementation-design boundary for CR154 before Story LLD work starts. This feature owns the first-wave shared reliability gate contract, strategy adapter mapping, gate-specific evidence policies, admission default policy and compatibility hooks for CR151 / CR152 / CR153.

This document is a planning artifact only. It does not approve LLD, source changes, test implementation, real runtime, real data, broker access or publishing.

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-03 | host-orchestrator | Initial CP4 feature design. Incorporates Story split review findings: Gate 5 explicit Story, Phase A runnable fixture schema ownership and Gate 6 tier resolution full-lld. |
| 0.2 | 2026-07-15 | host-orchestrator（inline meta-se） | CR-170 增量：为 21 个 mandatory evidence policy unit 增加五态判定、15/5/1 方向清单、Gate 1-5 局部消费、受保护 merge 与 T0-T3 admission 契约；明确 caller、T3 和 verifier 边界。 |
| 0.3 | 2026-07-15 | host-orchestrator（inline meta-dev） | CP5 评审补强：mandatory 精确到 applicable policy unit；conditional not-applicable complete N/A 使用 audit-only NR ref 且不设置 Gate floor；增加 public evidence→admission 端到端 fixture。 |

## CR-170 Increment: Canonical N/A Semantics Hardening

CR-170 复用 FEAT-15，不创建平行 Gate、平行 admission policy 或公共 schema。其目标是修复 canonical Gate 1-5 中 reason 字符串替代 mandatory evidence 的逃逸，并保证 Gate 6 不把 mandatory `NEEDS_REVIEW` 升级为无条件 `PASS`。

### Internal Contract Boundary

| 对象 | 精确职责 | 禁止行为 |
|---|---|---|
| `NaPolicySpec` | 记录 21 个 policy unit 的 Gate、evidence/reason keys、适用性、owner、baseline path type、hardening direction、complete-N/A disposition。 | 不暴露为 public API；不替代各 Gate 的数值、shape 或身份校验。 |
| `NaEvidenceDecision` | 返回五态之一：`PRESENT`、`MISSING`、`NA_WITH_COMPLETE_BOUNDARY`、`NA_WITH_INCOMPLETE_BOUNDARY`、`GENERIC_REASON_ESCAPE`。 | generic reason 不得成为 `PRESENT`；decision 不得直接产生 admission `PASS`。 |
| `n_a_boundaries[policy_id]` | 由当前 fixture/test caller 显式提供 `reason/owner/scope` 与 `release_profile` 或 `authorization_ref`。 | evaluator 合成 boundary/auth ref 的数量为 `0`；`authorization_ref` 只是 opaque audit pointer，不是凭据或运行授权。 |
| Gate 1-5 consumer | 在既有 Gate 局部把 decision 合并为 blocked/review claim 与 status floor；floor 只由 applicable mandatory unit 产生。conditional not-applicable complete N/A 只附加 `status=NEEDS_REVIEW` 的 audit-only ref，floor=None。 | 不修改全局 `_has_na_reason` 布尔语义；不让完整 N/A 产生 Gate `PASS`；不让 audit-only ref 单独抬升 Gate summary。 |
| Gate 6 merge/resolver | 先保护 `build_shared_gate_summary` / `evaluate_shared_contract` 既有 worst-state；仅硬化 `resolve_admission_policy` 对 Gate summary `NEEDS_REVIEW` 的 T0/T1/T2 处理。 | 不把 Gate ID 本身解释为全部 unit mandatory；受保护 merge 通过回归时 production diff 必须为 `0`；T3 early-return production diff 必须为 `0`。 |

### Inventory Direction Contract

| baseline path type | hardening direction | 数量 | 回归方向 |
|---|---|---:|---|
| 现有 reason escape | stricter | 15 | 原可放行路径必须变为 `NEEDS_REVIEW/BLOCKED`，无条件 `PASS=0`。 |
| 原 missing 即 blocked、现引入完整边界 | controlled-widening | 5 | 完整且适用的 N/A 可从 `BLOCKED` 变为可审计 `NEEDS_REVIEW`，但 T1/T2 必须 `BLOCKED` 且 `PASS=0`。 |
| 固定数值/来源阻断 | preserve | 1（G1-P06） | complete N/A 仍禁止，保持 `BLOCKED`。 |

21 项 exact mapping 由 S01 LLD 冻结并作为单一实现清单；任何数量变化都必须回退 CP3 或以 design delta 重开。

### Admission Tier Contract

| Tier | mandatory `NEEDS_REVIEW` | 兼容要求 |
|---|---|---|
| T0 / OPT_IN | `NEEDS_REVIEW` | 仅允许 fixture/static 诊断，不得宣称 admission PASS。 |
| T1 / DEFAULT_REQUIRED | `BLOCKED` | source rule 必须可审计。 |
| T2 / RELEASE_BLOCKING | `BLOCKED` | release wording 不得升级。 |
| T3 / NOT_AUTHORIZED | 保持现有 `BLOCKED + NOT_AUTHORIZED` | 只做 1/1 回归，生产代码修改数 `0`。 |

### CR-170 Story Ownership

| Story | Owner | 主要写入面 | 设计义务 |
|---|---|---|---|
| CR170-S01 | policy inventory / five-state contract | `engine/reliability_na_policy.py`、对应 unit tests | 21/21 exact rows、15/5/1、caller contract、deterministic reason IDs。 |
| CR170-S02 | Gate 1-5 consumers | canonical Gate module + Gate tests | 5/5 Gate、Gate1 三层断言、complete N/A 只到 review、generic escape PASS=0。 |
| CR170-S03 | protected merge / admission tiers | canonical Gate module + resolver tests | merge diff=0 when regression passes；T0/T1/T2 hardening；T3 diff=0。 |
| CR170-S04 | compatibility / claim closure | CR170 regression tests | public break=0、adapter 2/2、CR155/Stage3/real-op claim ceiling。 |

### CR-170 Failure and Rollback

| 触发 | 处理 |
|---|---|
| controlled-widening unit 产生 Gate/admission `PASS` | 立即回退该 unit 到历史 `BLOCKED`，不得以 waiver 放行。 |
| protected merge 或 T3 回归失败 | 停止实现并路由 `NEEDS_DESIGN_CLARIFICATION`；未获 design delta 前不得修改生产路径。 |
| caller 未提供完整 boundary | 判为 `NA_WITH_INCOMPLETE_BOUNDARY`，适用 mandatory unit 非 PASS。 |
| future aggregate/real caller 需要写 boundary | 由后续 CR 冻结 writer contract；CR-170 不预授权真实 caller。 |

## Feature Boundary

| Area | In Scope | Out of Scope |
|---|---|---|
| Shared contract | Gate summary, artifact refs, four-state status, blocked claims, release-blocking reason, first runnable fixture schema. | Runtime execution, live data reads, provider integration. |
| Strategy adapters | Multifactor, ML and event-driven mapping into the shared contract. | Replacing CR151 / CR152 / CR153 source semantics before LLD approval. |
| Gate evidence policies | Gate 1 statistical artifacts, Gate 2 CV governance, Gate 3 PIT universe, Gate 4 capacity/impact/liquidity, Gate 5 regime/attribution/reconciliation slots. | Real statistical calibration, real TCA, real universe construction, real reconciliation. |
| Admission default policy | Tier resolver and release wording semantics for opt-in / default-required / release-blocking / not-authorized. | Paper/live/trading readiness, broker readiness, true release execution. |
| Compatibility | CR153 `universe_pit_audit` lifecycle, CR152 ML-only n/a policy, CR151/152/153 admission summary consumption. | Deleting existing CR153 slot or silently changing historical CR semantics. |

## Story Split Decisions

| Review Finding | CP4 Treatment |
|---|---|
| Gate 5 was not explicit in HLD Phase B. | Add `CR154-S06-regime-attribution-reconciliation-slots` as a first-wave full-lld Story. |
| Phase A/B fixture schema ownership was ambiguous. | Put shared fixture schema and first runnable fixture in `CR154-S01-shared-gate-contract-fixture-skeleton`; Gate-specific fixtures extend it later. |
| Gate 6 tier resolution was under-specified. | Add `CR154-S07-admission-default-policy-tier-resolution` as a full-lld Story with resolver function/config boundary, fail-closed unknown profile behavior and release wording AC. |
| B1 statistical artifact workload is large. | Keep as one Story but require CP5 LLD to split adapter subtasks by strategy family and include 12 artifact fields, severity mapping and propagation cases. |

## Shared Contract Shape For CP5

CP5 LLD must define concrete names and serialization, but the following conceptual fields are mandatory:

| Field Family | Required Semantics |
|---|---|
| Gate identity | `gate_id`, `gate_name`, `strategy_class`, `release_profile`, `risk_level`, `evidence_completeness`. |
| Status | `PASS`, `FAIL`, `NEEDS_REVIEW`, `BLOCKED`, plus `n/a-with-reason` at artifact level. |
| Evidence refs | Structured refs with `artifact_type`, `ref`, `source_cr`, `owner_gate`, `status`, `n/a_reason`. |
| Blocked claims | Claim id, reason, source gate, release wording impact and unlock condition. |
| Release blocking | `release_blocking_reason`, gate mode and user-visible wording. |
| Forbidden operations | Counters for real lake/NAS/provider/runtime/broker/credential/feed/order/store/catalog/registry/publish operations; any nonzero count is `BLOCKED`. |

## Gate-Specific Design Requirements

| Gate | Story | Required CP5 Design Content |
|---|---|---|
| Gate 1 Backtest trap / statistical reliability | CR154-S02 | Multiple-testing correction refs, FDR/BH, WRC/SPA, PBO/CSCV, DSR/deflation, trial counts, OOS refs, purge/embargo refs, survivorship refs, impact/capacity refs, blocked claims and severity mapping by release profile. |
| Gate 2 CV governance | CR154-S03 | Shared walk-forward / OOS / purged-embargo contract and strategy-specific adapter mapping without copying CR152 ML-only semantics into multifactor/event strategies. |
| Gate 3 PIT universe | CR154-S04 | Survivorship-free / PIT universe gate, CR153 `universe_pit_audit` delegated source lifecycle, no real universe construction. |
| Gate 4 Capacity / impact / liquidity | CR154-S05 | ADV participation, capacity dollars, cost-underestimation status, liquidity sizing refs, `impact_model_family` enum: `square_root`, `almgren_chriss`, `gatheral`, `custom`, `n/a-with-reason`. |
| Gate 5 Regime / attribution / reconciliation | CR154-S06 | Structured slots, refs, status and `n/a-with-reason`; no runtime/broker reconciliation. |
| Gate 6 Admission default policy | CR154-S07 | Tier resolver from HLD §8, unknown profile fail-closed, release wording and compatibility behavior. |

## Follow-Through Hooks

| Hook ID | Feature Treatment |
|---|---|
| FT-CR154-CP5-001-WRC-SPA-SEVERITY | Owned by S02; CP5 must define missing WRC/SPA severity by release profile and claim type. |
| FT-CR154-CP5-002-CROSS-GATE-PROPAGATION | Owned by S02/S04/S05; CP5 fixtures must prove Gate 3/4 blocked states propagate into Gate 1 refs, blocked claims and release-blocking reason. |
| FT-CR154-CP5-003-NUMERIC-THRESHOLDS | Owned by S02/S05/S07; CP5 must define defaults, config ownership or explicit `n/a-with-reason` policy for DSR/PBO/capacity/risk-level thresholds. |
| FT-CR154-CP5-004-MF-GAP-2-6-7-DEFERRED | Owned by S08; CP5 mapping must preserve that MF-GAP-2/6/7 are deferred to factor-evaluation follow-up CR. |
| FT-CR154-CP5-005-REQ-ANCHOR-PRESERVATION | Owned by all Stories; CP5 must preserve HLD §11 UC/REQ mapping or route product-baseline refresh before implementation. |

## Integration Contract

| Caller / Neighbor | Direction | Contract |
|---|---|---|
| CR151 statistical gate | Consumed by CR154 adapters | CR154 must consume statistical refs and status; it must not weaken CR151 fail-closed behavior. |
| CR152 ML gate | Consumed by CR154 ML adapter | CR154 may mark ML-only fields for non-ML strategies as `n/a-with-reason`; unfinished ML method work remains CR152 follow-up. |
| CR153 event gate | Consumed by CR154 event adapter | CR153 `universe_pit_audit` remains a source slot; CR154 owns shared PIT release-blocking wording. |
| `StrategyAdmissionPackage` | Consumes CR154 summary | Package gets summary, blocked claims and release wording only; it does not gain runtime or trading authority. |

## Failure Paths

| Failure | Required Behavior |
|---|---|
| Missing mandatory artifact | `BLOCKED` for release-blocking profiles or `NEEDS_REVIEW` only where tier table explicitly allows it. |
| Unknown release profile | Fail closed until classified. |
| Nonzero forbidden operation counter | `BLOCKED`; cannot be waived in CP5/CP6/CP7. |
| Strategy-specific method not applicable | Must be explicit `n/a-with-reason`; omission is not acceptable. |
| Gate 3/4 blocked but Gate 1 refs clean | CP5 must treat this as cross-gate propagation failure. |

## Not Authorized

- No LLD approval before CP5.
- No source implementation or test implementation in CP4.
- No real lake read/write, NAS read/write/sync/restore, provider fetch, runtime, QMT, MiniQMT, xtquant, simulation, paper, live, trading, broker, credential, feed, order, reconciliation, store, catalog, registry, publish, external framework or Git remote action.
