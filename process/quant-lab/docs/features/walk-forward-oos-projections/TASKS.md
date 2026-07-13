---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-04"
---

# FEAT-166-04 任务清单

| Task ID | Story | 任务 | 文件 | 依赖 / DoD |
|---|---|---|---|---|
| F166-04-K01 | S04 | 建立 projection-only module 与 identity guard | `engine/walk_forward_oos_projections.py` | S03；mismatch blocked。 |
| F166-04-K02 | S04 | 接入 statistical gate | `engine/strategy_admission_statistical_gate.py` | K01；legacy compatibility，policy owner 不变。 |
| F166-04-K03 | S04 | 接入 reliability Gate 2 | `engine/cross_strategy_reliability_gates.py` | K01；same identity/reasons。 |
| F166-04-K04 | S04 | 接入 admission package | `engine/strategy_admission_package.py` | K01；worse-only，runtime flags 不变。 |
| F166-04-K05 | S04/S05 | 3 consumers + CR155 regression tests | projection/regression tests | K01-K04；3/3，promotion=0。 |

S04 在 W4 独占以上生产文件；CP5 批准前禁止实现或修改测试。
