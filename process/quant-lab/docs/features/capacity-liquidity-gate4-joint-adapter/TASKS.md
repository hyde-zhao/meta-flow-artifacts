---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-03"
---

# FEAT-169-03 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 建立 strict joint adapter 与 public callable double 任务。 |

| TASK-ID | Story | 顺序 | 动作 | 输出 | 完成准则 |
|---|---|---:|---|---|---|
| G4J-K01 | S04 | 1 | 定义 outcome/reason/validator Protocol | `engine/capacity_liquidity_gate4_projection.py` | keyword-only DI 与 immutable outcome。 |
| G4J-K02 | S04 | 2 | 实现 type/header/7-key/reason prechecks | 同上 | 失败 canonical calls=0。 |
| G4J-K03 | S04 | 3 | 调 public validator + local postcondition | 同上 | candidate-release；worse-state only。 |
| G4J-K04 | S04/S05 | 4 | 创建 adapter tests / source guards | `tests/research/test_capacity_liquidity_gate4_projection.py` | G4J-T01..T09。 |
