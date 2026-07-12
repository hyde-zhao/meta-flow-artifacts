# FEAT-24 TEST-PLAN

| 层级 | 场景 | 目标 |
|---|---|---|
| unit | schema/status/reason/domain | valid/invalid branch 100% |
| contract | family ref/hash/raw count/membership | binding coverage 100%，difference=0 |
| deterministic | same payload 10 runs | 1 hash |
| negative | missing、mismatch、NaN/Inf、orphan | false PASS=0 |

所有测试使用 local synthetic fixtures；禁止真实数据/runtime/network。每个公共接口至少一条正向与一条失败测试。

