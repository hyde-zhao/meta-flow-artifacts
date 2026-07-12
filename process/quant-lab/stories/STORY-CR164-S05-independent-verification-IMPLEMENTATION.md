# S05 Implementation — QAC / Authorization / CR155

## 实现前置检查

S01-S04 implemented；verification owns tests only and does not modify `engine/**`。

## 实现对象与映射

- `test_statistical_evidence_qac.py`: binding/count/hash/orphan/3-consumer/negative denominator。
- `test_statistical_evidence_authorization.py`: AST import/write/env guard and operation counters。
- `test_statistical_evidence_cr155_regression.py`: missing native lineage remains blocked even when unrelated future summary passes。
- `tests/PROVENANCE.yaml`: 7/7 CR164 test paths registered under research domain。

## 验证结果

CR164 + CR151/CR163 related subset: 76 passed。Full suite: 1935 passed, 14 failed; all 14 are pre-existing workflow/artifact hygiene and flat CR163 test debt, not CR164 functional failures. No design delta or waiver。

