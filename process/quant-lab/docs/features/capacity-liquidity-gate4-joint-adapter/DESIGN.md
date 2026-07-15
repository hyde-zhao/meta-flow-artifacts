---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-03"
feature_name: "Strict C3+C4 Gate4 Fixture Adapter"
related_stories: ["CR169-S04", "CR169-S05"]
---

# FEAT-169-03 Strict C3+C4 Gate4 Fixture Adapter 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se/meta-dev | 冻结 7-key allowlist、reason-key/extra-key 拒绝、public validator DI 与 fixture-only postcondition。 |

## 1. 集成契约

唯一入口 `evaluate_c3_c4_gate4_fixture_compatibility` 只接收 self-validated `economic_cost@v1`、`capacity_liquidity@v1`、各自 attachment 与显式 correlation context。顺序固定：type/schema/self-hash → 13-field exact match → 构造 exact 7-key payload → reason/extra-key presence scan → public canonical call → local postcondition。

精确 payload：

- C3：`impact_model_family`、`impact_model_ref`、`cost_underestimation_status`、`no_real_tca_claim`；
- C4：`adv_participation_ref`、`capacity_dollars_ref`、`liquidity_sizing_refs`。

任何 `*_na_reason`、`*_n_a_reason`、`na_reason`、`n_a_reason`、absent placeholder、聚合/admission 字段或第 8 个 payload key 均在 canonical 前 REJECTED，call count=0。

## 2. Public callable 与 DI

生产默认依赖公开 callable：

```python
validate_gate4_capacity_impact(
    evidence: Mapping[str, Any],
    *,
    release_profile: str = "candidate-release",
    operation_counts: Mapping[str, Any] | None = None,
) -> ReliabilityGateSummary
```

adapter 每次显式传 `release_profile="candidate-release"`。LLD 以 `Protocol.__call__` 表达 keyword-only DI；测试使用实现该 Protocol 的 callable double，不 monkeypatch canonical module，不依赖 `_has_na_reason` 等私有 helper。

## 3. Postcondition 与结果上限

只有本地前提全部成立且 canonical 返回 PASS 时，才返回唯一 `gate4_fixture_contract_pass`。该 outcome 固定携带 `aggregate_admission_pass=false`、`capacity_scalable_claim=false`、`real_capacity_ready=false`、`stage3_entry_ready=false`。canonical 非 PASS 时原样降级为 BLOCKED/NEEDS_REVIEW/FAIL；不得 worse→better。若 double 模拟 unexpected PASS 但前提/claim 不变量不成立，返回 `REJECTED/gate4_fixture_postcondition_violation`。

## 4. 不修改与回退

- `engine/cross_strategy_reliability_gates.py`、`engine/economic_cost_gate4_projection.py`、`engine/strategy_admission_package.py` 为 forbidden-write。
- 不调用 aggregate orchestration；不创建 direct arbitrary mapping API。
- 回退是删除/停用 CR169 local adapter，同时保留 C4 component typed present/unavailable 合同；CR168 absent-C4 adapter 不变。

## 5. Gotchas

- C4 present path 不允许 N/A reason；三 refs 必须 typed present。
- canonical PASS 只证明 fixture 7-field contract，不证明容量可扩、admission 或 Stage3。
- 不以 runtime import 私有 helper“保持一致”；候选语义只通过公开输入/输出测试锁定。
