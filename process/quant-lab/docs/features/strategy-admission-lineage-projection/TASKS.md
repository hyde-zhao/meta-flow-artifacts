---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-22"
---

# Feature Tasks: Strategy-admission Lineage Projection

| TASK-ID | 顺序 | Story | 任务 | 输出文件 | Ownership | 验证 | 状态 |
|---|---:|---|---|---|---|---|---|
| TASK-CR163-22-01 | 1 | S04 | 冻结 projection DTO 与 availability/status mapping | S04 LLD | design | TP22-01..04 | pending |
| TASK-CR163-22-02 | 2 | S04 | 规划 CR151 consumer adapter | `engine/strategy_admission_statistical_gate.py` | primary S04 | TP22-01/02/03 | pending |
| TASK-CR163-22-03 | 3 | S04 | 规划 CR154 consumer adapter | `engine/cross_strategy_reliability_gates.py` | primary S04 | TP22-01/03/04 | pending |
| TASK-CR163-22-04 | 4 | S04 | 规划 package attach/refs/status precedence | `engine/strategy_admission_package.py` | primary S04 | TP22 all | pending |
| TASK-CR163-22-05 | 5 | S05 | 汇总 consumer/CR155/claim-ceiling fixtures | consumer regression tests | primary S05 | TP22 all | pending |

## 阻塞项

无；不得为了满足既有 effective-count policy 扩大 CR163 范围。
