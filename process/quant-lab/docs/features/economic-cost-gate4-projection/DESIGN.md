---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-02"
feature_name: "Economic Cost Gate4 Projection"
source_hld: "process/archive/design-cr-docs/HLD-ECONOMIC-COST-IMPACT-EVIDENCE.md"
source_adr: "process/archive/design-cr-docs/ARCHITECTURE-DECISION-ECONOMIC-COST-IMPACT-EVIDENCE.md"
related_stories: ["CR168-S04", "CR168-S05"]
---

# FEAT-168-02 Economic Cost Gate4 Projection 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 冻结 CR168 唯一 projection adapter、4-key allowlist、8-key presence denylist、candidate-release 与双 postcondition reason。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：锁定三个 canonical C4 missing claim ID，并把 double 机制定为 public-callable 的窄依赖注入，不使用 monkeypatch 或 private helper。 |

## 1. 目标与威胁模型

Gate4 是 C3+C4 联合门禁。canonical validator 当前允许 C4 ref absent + N/A reason 绕过 missing claim，因此 CR168 的第一个 caller 必须在 adapter 内封闭 reason escape，同时不得修改 canonical 或 aggregate。

| Goal | 精确值 |
|---|---:|
| adapter/call site | 1/1 |
| C3 output keys | 4/4 |
| forbidden keys rejected | 8/8 |
| escape canonical calls | 0 |
| safe absent PASS exposed | 0 |
| canonical/aggregate source changes | 0/0 |

## 2. 模块、接口与依赖

| 文件 | 动作 | 职责 |
|---|---|---|
| `engine/economic_cost_gate4_projection.py` | 创建 | typed precheck、strict payload build、canonical public call、postcondition、outcome。 |
| `engine/cross_strategy_reliability_gates.py` | 只读 | public `validate_gate4_capacity_impact`；修改数 0。 |

| Interface | 输入 | 输出 | 错误 |
|---|---|---|---|
| `project_economic_cost_to_gate4(evidence, c4_availability, operation_counts, *, gate4_validator=None)` | present `EconomicCostEvidenceV1`、C4 unavailable marker、zero counts；仅测试可注入 public validator callable | `C3Gate4ProjectionOutcome` | pre-call blocked/rejected；post-call blocked；never aggregate |
| `_build_gate4_payload(evidence)` | typed evidence only | 精确 4-key dict | arbitrary mapping passthrough 禁止 |

依赖只允许 adapter→C3 typed value、adapter→canonical public validator；禁止 canonical→adapter、runtime import `_has_na_reason`、adapter→aggregate/package/C4 calculator。

## 3. Pre-call 合同

允许 payload key 精确为：`impact_model_family`、`impact_model_ref`、`cost_underestimation_status`、`no_real_tca_claim`。三个 C4 ref key完全省略。

Forbidden key 按 presence 拒绝，空值也拒绝：

1. `adv_participation_ref_na_reason`
2. `adv_participation_ref_n_a_reason`
3. `capacity_dollars_ref_na_reason`
4. `capacity_dollars_ref_n_a_reason`
5. `liquidity_sizing_refs_na_reason`
6. `liquidity_sizing_refs_n_a_reason`
7. `na_reason`
8. `n_a_reason`

| 条件 | canonical calls | Outcome |
|---|---:|---|
| present square_root + C4 unavailable + clean + zero counts | 1 | postcondition |
| C3 非 present/unsupported | 0 | BLOCKED |
| C4 present/C4 refs | 0 | REJECTED_OUT_OF_SCOPE |
| 任一 forbidden key | 0 | BLOCKED `gate4_reason_escape_rejected` |
| 任一 operation count 非 0 | 0 | BLOCKED `external_operation_forbidden` |

## 4. Canonical call 与 Postcondition

- `release_profile` 在 adapter 内固定为 `candidate-release`，caller 无参数覆盖面。
- 合法 safe-absent 结果必须 `status=BLOCKED` 且 blocked claims 精确包含 `adv_participation_missing`、`capacity_dollars_missing`、`liquidity_sizing_missing`；测试不得以模糊的 “three claims” 代替这三个 ID。
- canonical PASS → adapter BLOCKED `gate4_unexpected_pass`。
- canonical 非 PASS 但缺任一预期 claim → adapter BLOCKED `gate4_postcondition_violation`。
- 两类内部合同错误都不外传 canonical status，aggregate calls=0。

SIM-05 一律采用**窄依赖注入**：production caller 不传 `gate4_validator`，adapter 默认调用 public `validate_gate4_capacity_impact`；测试把只返回 public `ReliabilityGateSummary` 的 fake callable 作为 keyword-only dependency 传入。禁止 monkeypatch canonical module，禁止 import/复制 `_has_na_reason` 或其他 private helper，fake 只模拟返回值而不模拟 canonical 实现。真实 integration 另测 safe-absent canonical BLOCKED。

## 5. 演进、回退与 Gotchas

- CR168 只设计 C3 present + C4 unavailable。FU-005 决定 C4 present 后是扩展本 adapter 还是新增 C4-owned projection；本 Feature 不锁死。
- 出现 adapter 外 direct caller、aggregate 接入或全局保证诉求时，必须进入 FU-007/独立 remediation，不能引用本 Feature 宣称 global fixed。
- 任一 payload 逃逸、PASS 外传或 canonical source 修改，回退 component-only 并重开范围/架构门禁。
- Gotcha：不能把 canonical 当前 permissive 行为锁成 golden；只锁 adapter 安全后果。
- Gotcha：C4 `typed_unavailable` 在 flat payload 的安全表达是 ref/reason key 都 absent，不是 `*_na_reason`。
