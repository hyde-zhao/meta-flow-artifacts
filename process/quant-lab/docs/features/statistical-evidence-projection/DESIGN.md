# FEAT-27 Conservative Aggregation and Projection — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host inline meta-se | 初始实现设计。 |

Aggregator 只消费经验证 MethodEvidence refs，不重算。claim-specific mandatory set 写入 summary，不得按 available set 缩小。决策表：任一 blocked→BLOCKED；否则任一 fail→FAIL；否则任一 unavailable→TYPED_UNAVAILABLE；全部 mandatory pass→PASS。orphan/missing ref→BLOCKED。

Projection 复用 CR151、CR154、admission package 三 surfaces；status 只可持平或变差。UC-58 接入实现，UC-59/60 只验证 compatibility contract；不创建 real adapter 或新 gate。raw-count DSR 不填 effective fields。

## Gotchas

- 不得把 informational 方法纳入或移出 mandatory set 来改变结果。
- consumer 不得自己解释 child method details 后改善状态。
- 3/3 projection coverage 不等于 3/3 adapter implementation。

