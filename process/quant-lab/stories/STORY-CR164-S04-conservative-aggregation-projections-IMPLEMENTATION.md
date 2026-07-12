# S04 Implementation — Conservative Aggregation / Projections

## 实现前置检查

S01-S03 APIs available；existing consumer modules are the approved owners。

## 实现对象与映射

`aggregate_statistical_evidence` implements explicit mandatory denominator and `BLOCKED > FAIL > TYPED_UNAVAILABLE > PASS`。CR151、CR154、admission package 新增 thin projection/attach functions；serialized summaries cannot be positive truth；status only stays or worsens；Gate IDs unchanged。

`tests/research/test_statistical_evidence_projection.py` covers disagreement、3 consumers、untrusted serialization、status worsening、effective ceiling和 no-new-gate。

## 边界

No parallel gate、consumer recomputation、lineage write、runtime authorization or design delta。

