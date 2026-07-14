# CR168-S04 Implementation — Gate4 Projection 局部安全封闭

S04 新建唯一的 `project_economic_cost_to_gate4` adapter。它只重建四个 C3 键：`impact_model_family`、`impact_model_ref`、`cost_underestimation_status`、`no_real_tca_claim`；三个 C4 ref 和所有 C4/general N/A reason 键均不进入 payload。

在 canonical 调用前，8 个 reason key 按 key presence（空值也算）拒绝，C4 ref key presence、C4 present、非零 forbidden operation 或不可用 C3 都使 canonical calls=0。clean C3 + C4 `typed_unavailable` 路径固定调用 public `validate_gate4_capacity_impact(..., release_profile="candidate-release")`，只接受精确的 `adv_participation_missing`、`capacity_dollars_missing`、`liquidity_sizing_missing` 三项 BLOCKED claims；canonical PASS 与 non-PASS claim mismatch 分别本地封闭为 `gate4_unexpected_pass` 和 `gate4_postcondition_violation`。

测试的 fake 仅经 keyword-only `gate4_validator=` 注入并返回 public `ReliabilityGateSummary`。未使用 monkeypatch、canonical private helper、aggregate orchestration；未修改 `engine/cross_strategy_reliability_gates.py` 或 `engine/strategy_admission_package.py`。

## 验证

- `py_compile engine/economic_cost_gate4_projection.py tests/research/test_economic_cost_gate4_projection.py`：PASS。
- S01–S04/C2 targeted pytest：55 passed in 0.18s。
- B01 real canonical safe-absent、B02 8/8 reason-key escape、C4 present、nonzero operation、public double PASS、public double missing claims，以及 private/aggregate static guard：PASS。
- canonical/aggregate source diff=0/0；`git diff --check`：PASS。
