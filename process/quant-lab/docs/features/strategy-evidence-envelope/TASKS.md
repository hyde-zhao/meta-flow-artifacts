---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-01"
---

# FEAT-166-01 任务清单

| Task ID | Story | 任务 | 文件 | 依赖 | 完成准则 |
|---|---|---|---|---|---|
| F166-01-K01 | S01 | 建立 neutral canonical/availability contracts | `engine/strategy_evidence.py` | 无 | 显式 domain、finite-only、immutable types。 |
| F166-01-K02 | S01 | 建立 stable envelope/static catalog | `engine/strategy_evidence.py` | K01 | active/reserved/unknown decision table 完整。 |
| F166-01-K03 | S01 | 建立 C2 input/evidence value shape | `engine/walk_forward_oos_evidence.py` | K01-K02 | 7/7 字段族可表达，无计算/I/O。 |
| F166-01-K04 | S01 | C1 compatibility wrapper/re-export | `engine/statistical_evidence.py` | K01 | API/bytes/default domain/hash 差异=0。 |
| F166-01-K05 | S01/S05 | contract/golden/catalog/tamper tests | 两个 contract test 文件 | K01-K04 | TEST-PLAN 6/6；forbidden operations=0。 |

执行顺序：K01→K02→K03/K04→K05。CP5 批准前全部为设计任务，禁止修改所列源代码/测试。
