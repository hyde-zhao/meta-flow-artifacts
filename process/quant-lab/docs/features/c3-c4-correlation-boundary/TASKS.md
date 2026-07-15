---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-02"
---

# FEAT-169-02 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 建立 header/view/correlation validation 任务。 |

| TASK-ID | Story | 动作 | 文件 | 完成准则 |
|---|---|---|---|---|
| CH-K01 | S01 | 定义 header/context immutable values | `engine/capacity_liquidity_evidence.py` | exact fields=13。 |
| CH-K02 | S04 | 构造 C3/C4 header views 并 exact compare | `engine/capacity_liquidity_gate4_projection.py` | mismatch canonical calls=0。 |
| CH-K03 | S04/S05 | 创建 13-field matrix tests | `tests/research/test_capacity_liquidity_gate4_projection.py` | CH-T01..T06。 |
