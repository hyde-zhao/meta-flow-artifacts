---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-03"
---

# FEAT-166-03 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 定义 fold 重算、分母、determinism、自校验与非 present 前置验证。 |

| Test ID | 输入 | 断言 |
|---|---|---|
| F166-03-T01 | complete validated daily | fold/metric decisions、counts、pass rate 重算差异=0。 |
| F166-03-T02 | mixed pass/fail folds | denominator=declared；FAIL fold 不被过滤。 |
| F166-03-T03 | missing/invalid observed fold | non-present，pass_rate=null。 |
| F166-03-T04 | validated ML mapping | same component schema；不训练模型。 |
| F166-03-T05 | repeated normalized input | 10 runs→1 component hash 与 1 envelope hash。 |
| F166-03-T06 | tampered result/old hash | self-validation 100% blocked。 |
| F166-03-T07 | unavailable/blocked precondition | producer 不进入 calculation；false present=0。 |
| F166-03-T08 | operation instrumentation | file/env/network/provider/runtime calls=0。 |

测试文件：`tests/research/test_walk_forward_oos_producer.py`。通过标准 8/8、recompute difference=0、10→1、false present=0、operation=0。
