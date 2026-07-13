---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-05"
---

# FEAT-166-05 任务清单

| Task ID | Story | 任务 | 依赖 | 完成准则 |
|---|---|---|---|---|
| F166-05-K01 | S05 | 建立 daily/ML positive 与 8 类 negative fixture catalog | S01-S04 | 仅合成事实；真实数据样本=0。 |
| F166-05-K02 | S05 | 建立 QAC 汇总测试 | K01 | 12/12 QAC 可机器断言。 |
| F166-05-K03 | S05 | 建立 authorization/claim scan | K01 | dereference=0、forbidden=0、Stage flags 4/4。 |
| F166-05-K04 | S05 | 建立 CR155/CR165 regression attribution | K01 | CR155 promotion=0；触达历史失败逐项归因率=100%。 |
| F166-05-K05 | S05/CP7 | 分层执行并生成 evidence index/report | K02-K04 | 11/11 scenarios、12/12 QAC、CR166 failure=0。 |

S05 只在 S01-S04 实现完成并通过 CP6 后执行；本轮 CP5 前只冻结设计，不运行测试。
