---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-02"
---

# FEAT-166-02 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 定义 input、temporal、leakage、metric、lineage、authorization 与 adapter 验证。 |

| Test ID | 场景 | 预期 |
|---|---|---|
| F166-02-T01 | daily complete mapping | 7/7 字段族 validated；不改 legacy object。 |
| F166-02-T02 | ML complete mapping | same common semantics；无 training/registry/runtime。 |
| F166-02-T03 | missing/empty/count mismatch fold | non-present；declared fold 不被缩小。 |
| F166-02-T04 | reversed/illegal temporal bounds | 3/3 temporal/leakage class 中 temporal blocked，fold/field reason 非空。 |
| F166-02-T05 | missing/insufficient purge | overlap 时 blocked；ref-only 不通过。 |
| F166-02-T06 | embargo one-below/exact | one-below blocked，exact 可继续。 |
| F166-02-T07 | missing/NaN/Inf metric | 3/3 fail-closed，present non-finite=0。 |
| F166-02-T08 | lineage missing/mismatch/orphan | missing unavailable；contradiction blocked；orphan=0。 |
| F166-02-T09 | unauthorized ref/nonzero counter | dereference=0，blocked，所有实际 forbidden calls=0。 |
| F166-02-T10 | event applicability | N/A 1/1；event producer/fixture/feed access=0。 |

测试文件：`tests/research/test_walk_forward_oos_validation.py`；S05 另做 QAC 汇总。通过标准 10/10 groups，daily/ML=2/2，P0 fail-closed false PASS=0。
