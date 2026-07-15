---
feature_id: "FEAT-15"
change_id: "CR-170"
baseline_change_id: "CR-154"
status: "ready-for-cp5-review"
version: "0.3"
created_at: "2026-07-03T10:40:00+08:00"
owner: "host-orchestrator"
implementation_allowed: false
---

# Cross-Strategy Reliability Gates Test Plan

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-03 | host-orchestrator | CR-154 初始测试计划。 |
| 0.2 | 2026-07-15 | host-orchestrator（inline meta-se） | CR-170 增量：新增 21-unit inventory、五态、15/5/1 双向回归、Gate1 三层断言、受保护 merge、T0-T3 与 adapter/claim 回归。 |
| 0.3 | 2026-07-15 | host-orchestrator（inline meta-dev） | CP5 评审补强：增加 conditional not-applicable audit-only ref/no-floor 断言、unit-level mandatory 表述与 public-callable evidence→admission 端到端 fixture。 |

## CR-170 Validation Matrix

| 测试组 | Owner Story | 精确覆盖 | 通过条件 |
|---|---|---|---|
| Policy inventory | S01 | 21/21；Gate 分布 6/6/1/5/3；baseline path type、direction、disposition 全填 | exact IDs 唯一；方向 15/5/1；五态 5/5。 |
| Boundary classifier | S01 | complete 4/4、incomplete、generic reason、missing、present；fixture/test caller | evaluator synthesis=0；generic/incomplete 非 `PRESENT`。 |
| Directional Gate regression | S02 | stricter 15、controlled-widening 5、preserve 1 | 收紧组 PASS=0；放宽组只到 NR 且 T1/T2 BLOCKED；G1-P06 BLOCKED。 |
| Conditional N/A audit boundary | S02 | complete not-applicable 1/1 + incomplete/generic negatives | audit ref status=NR、mandatory claim=0、status floor=None、Gate status elevation=0；invalid path 不得放行。 |
| Gate1 masked escape | S02 | field decision、mandatory claim、final worst-state | 3/3 独立断言，不以最终 status 掩盖字段缺陷。 |
| Protected merge | S03 | summary 中任一 Gate NR 的传播 | 1/1；通过时 production diff=0。 |
| Admission tiers | S03 | T0/T1/T2/T3 | NR/BLOCKED/BLOCKED/NOT_AUTHORIZED；T3 production diff=0。 |
| Compatibility/claim | S04 | public signatures/Gate IDs、CR168 adapter、CR169 adapter、CR155、Stage3/aggregate/real-op | 100%、2/2、promotion=0、unauthorized ops=0。 |
| Public end-to-end | S04 | `n_a_boundaries`→Gate validator→shared merge→T1 resolver | Gate NR、merge NR、T1 BLOCKED；admission PASS=0；private helper calls=0。 |

### Exact regression groups

S01 LLD 必须给 21 个 policy unit 标注三列：`baseline_path_type`、`hardening_direction`、`complete_na_disposition`。S02 的参数化测试直接消费该 inventory，禁止维护第二份手写计数。

### Caller contract tests

- 当前可写 `n_a_boundaries` 的生产外 caller 只有 fixture/test builder，数量 `1` 类；生产 evaluator 只读且不得合成。
- `authorization_ref` 只接受非空 opaque audit pointer；测试不得读取 `.env`、credential、token 或外部授权系统。
- `reason/owner/scope` 任一缺失，或 `release_profile` 与当前调用不匹配且无合法 `authorization_ref`，必须为 `NA_WITH_INCOMPLETE_BOUNDARY`。

### T3 compatibility test

测试必须先调用现有 T3 early-return 基线，再应用 CR-170 变更；结果保持 `status=BLOCKED`、`mode=NOT_AUTHORIZED`，且 T3 分支生产 diff 为 `0`。

## Purpose

Define CP5/CP6/CP7 validation expectations for CR154 without authorizing implementation. All validation remains local/static/fixture-only unless a later human gate explicitly changes the boundary.

## Test Levels

| Level | Scope | Required Coverage |
|---|---|---|
| Contract unit tests | Shared summary, artifact refs, status enum, blocked claims. | PASS/FAIL/NEEDS_REVIEW/BLOCKED, `n/a-with-reason`, forbidden operation counters. |
| Gate policy tests | Gate 1-6 evidence policy behavior. | Missing mandatory evidence, release profile severity, fail-closed unknown profile, impact enum validation. |
| Adapter tests | Multifactor / ML / event-driven mapping. | Strategy-specific refs, ML-only n/a policy, CR153 `universe_pit_audit` delegation. |
| Integration tests | Admission package summary and wording. | Blocked claims, release-blocking reason, no runtime/trading readiness claims. |
| Regression tests | CR151 / CR152 / CR153 existing fixture behavior. | Existing first-wave gates remain consumable and no prior package-visible semantics are silently removed. |

## Fixture Matrix

| Fixture Group | Story Owner | Required Cases |
|---|---|---|
| Shared fixture skeleton | S01 | Minimal PASS, missing artifact `BLOCKED`, `NEEDS_REVIEW`, forbidden operation counter `BLOCKED`. |
| Statistical artifacts | S02 | Complete 12-slot evidence, missing WRC/SPA severity, missing PBO/DSR, trial count invalid, Gate 3/4 propagation. |
| CV governance | S03 | Walk-forward present, OOS split missing, purge/embargo missing, ML-specific CV mapped without forcing ML-only semantics on other strategies. |
| PIT universe | S04 | PIT pass, non-PIT blocked, CR153 slot delegated, CR153 slot absent with blocked claim. |
| Capacity / impact | S05 | Allowed impact enum, custom with rationale, invalid enum, cost-underestimation blocked, no-real-TCA wording. |
| Regime / attribution / reconciliation | S06 | Slot pass, slot n/a-with-reason, missing reason blocked/review per tier, no runtime reconciliation claim. |
| Admission tier | S07 | T0 opt-in, T1 default-required, T2 release-blocking, T3 not-authorized, unknown profile fail-closed. |
| Compatibility / wording | S08 | MF-GAP-2/6/7 deferred wording, CR151/152/153 summary compatibility, no overclaim release text. |

## Forbidden Validation

The following must not appear in CR154 tests unless a later authorization gate changes the boundary:

- Reading `.env`, secrets, accounts, tokens, session or broker credentials.
- Real lake / NAS / provider access.
- Real event feed, live listener, model registry, event store, feature store, prediction store or catalog pointer write.
- QMT / MiniQMT / xtquant / gateway runtime, simulation, paper, live, trading, broker query, submit or cancel.
- External framework clone/install/run or Git remote write.

## CP5 Acceptance Expectations

CP5 design evidence must show the planned tests before implementation starts. In particular:

- S01 must own fixture schema and the first runnable fixture.
- S02 must explicitly test WRC/SPA severity and Gate 3/4 propagation.
- S05 must test `square_root`, `almgren_chriss`, `gatheral`, `custom` and `n/a-with-reason`.
- S07 must test fail-closed unknown release profile.
- S08 must prove release wording cannot claim production, paper/live, broker, real data or trading readiness.
