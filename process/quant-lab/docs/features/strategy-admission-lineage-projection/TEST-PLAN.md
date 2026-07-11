---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-22"
---

# Feature Test Plan: Strategy-admission Lineage Projection

## 测试范围

| Scope | 场景 | 测试 | 量化出口 |
|---|---|---|---|
| TP22-01 | P03 | valid manifest/result projection through 3 consumers | present only if 5 check classes PASS；3/3 consumers receive same ref/raw count |
| TP22-02 | N01 | absent vs post-hoc/incomplete | uninstrumented 100% typed_unavailable；invalid 100% blocked |
| TP22-03 | B02 | effective/C1 ceiling | effective available claims=0；ref/method nonempty count=0；C1 computed=0 |
| TP22-04 | T01/manual conflict | tamper or count mismatch | 100% blocked；status never improves |
| TP22-05 | G01 | CR155 regression | blocked 1/1；paper_candidate=false；backfill=0 |

## Security / compatibility

- Verify runtime/data/credential/external operation counters all 0.
- Verify existing consumer public behavior remains backward fail-closed when projection absent.
- Verify no new gate ID/family is introduced.

## 手工验收

Review one table of all availability→consumer status mappings and confirm raw lineage readiness wording never states statistical proof/runtime readiness.

