---
status: "ready-for-cp5-review"
version: "0.2"
cr_id: "CR-166"
feature_id: "FEAT-166-01"
change_refs: ["CR-166", "CR-168"]
---

# FEAT-166-01 任务清单

| Task ID | Story | 任务 | 文件 | 依赖 | 完成准则 |
|---|---|---|---|---|---|
| F166-01-K01 | S01 | 建立 neutral canonical/availability contracts | `engine/strategy_evidence.py` | 无 | 显式 domain、finite-only、immutable types。 |
| F166-01-K02 | S01 | 建立 stable envelope/static catalog | `engine/strategy_evidence.py` | K01 | active/reserved/unknown decision table 完整。 |
| F166-01-K03 | S01 | 建立 C2 input/evidence value shape | `engine/walk_forward_oos_evidence.py` | K01-K02 | 7/7 字段族可表达，无计算/I/O。 |
| F166-01-K04 | S01 | C1 compatibility wrapper/re-export | `engine/statistical_evidence.py` | K01 | API/bytes/default domain/hash 差异=0。 |
| F166-01-K05 | S01/S05 | contract/golden/catalog/tamper tests | 两个 contract test 文件 | K01-K04 | TEST-PLAN 6/6；forbidden operations=0。 |
| F166-01-K06 | CR168-S03 | 激活 `economic_cost@v1` descriptor 并保持 capacity reserved | `engine/strategy_evidence.py` | CR168 S01/S02 | active schema=1；C1/C2 bytes/hash 差异=0。 |
| F166-01-K07 | CR168-S03/S05 | 验证 component/envelope identity 分域与 tamper | `tests/research/test_economic_cost_envelope_compatibility.py` | K06 | component distinct=1；envelope distinct=2；tamper false PASS=0。 |

原 CR166 顺序保持 K01→K02→K03/K04→K05；CR168 增量为 K06→K07，且依赖 CR168 S01/S02。CR168 CP5 批准前全部为设计任务，禁止修改所列源代码/测试。
