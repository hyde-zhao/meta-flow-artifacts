---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-01"
---

# FEAT-169-01 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 建立 S01/S02/S03/S05 的受控实现任务。 |

| TASK-ID | Story | 顺序 | 动作 | 输出 | 完成准则 |
|---|---|---:|---|---|---|
| CL-K01 | S01 | 1 | 创建 immutable input/header/issue/build-result contracts | `engine/capacity_liquidity_evidence.py` | schema/type/domain 与 N01..N12 冻结。 |
| CL-K02 | S01 | 2 | 创建 normalize/validate/hash/self-validation | 同上 | 12/12；10→1；identity 分域。 |
| CL-K03 | S02 | 3 | 创建 pure Decimal static proxy calculator | `engine/capacity_liquidity_calculator.py` | 公式、cap、minor-unit 断言精确。 |
| CL-K04 | S02 | 4 | 组合唯一 public producer | `engine/capacity_liquidity_evidence.py` | issue short-circuit；3 refs present。 |
| CL-K05 | S03 | 5 | 提供 neutral envelope attach 所需 public values | public exports | 不修改 C1/C2/C3 semantic hash。 |
| CL-K06 | S01/S02/S05 | 6 | 创建 contract/producer/negative tests | `tests/research/test_capacity_liquidity_*.py` | CL-T01..T10。 |

CP5 前禁止执行任务；任何 schema/method/hash 变化回 CP3。
