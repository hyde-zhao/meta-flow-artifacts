---
status: final-for-cp8
version: "1.0"
cr_id: "CR-173"
test_decision: PASS_WITH_RISK
updated_at: "2026-07-17T12:30:00+08:00"
---

# CR-173 Test Report

## 摘要

| 项目 | 结果 |
|---|---|
| 最终结论 | PASS_WITH_RISK |
| 最终 CP7 Story | 3/3 PASS |
| BLOCKER / HIGH 未关闭 findings | 0 / 0 |
| Waiver | 0 |
| 真实数据/runtime/publish/deploy 执行 | 0/0/0/0 |

## 执行结果

| 验证层 | 结果 | 证据 |
|---|---|---|
| S01 typed evidence/canonicalization | 60/60 PASS；相邻对抗 10/10 | `process/evidence/CR173-S01-contract-evidence-canonicalization.CP7R2.index.json` |
| S02 exact estimator | 87/87 PASS；exact PSD oracle 854/854；compile 3/3 | `process/evidence/CR173-S02-exact-spectral-estimator.CP7.index.json` |
| S03 golden/failure/authz | authorization 9/9；targeted 124/124；combined 379/379；compile 2/2 | `process/evidence/CR173-S03-golden-failure-boundary-verification.CP7R1.index.json` |
| 确定性 | golden 6/6 × repeat 3/3 | S03 CP7R1 evidence |
| 失败语义 | F01-F08 8/8；raw alias=0 | S03 CP7R1 evidence |
| public/claim 边界 | 8 production + 12 expected path 修改=0/0；public populatable/c1 computable=false/false | S03 CP7R1 result |

## 风险解释

`PASS_WITH_RISK` 不是测试失败或 waiver。它表示 fixture-only v1 已满足批准合同，同时保留两个知情边界：exact rational 的整数增长未建立 production SLA；participation ratio 不得解释为多重检验校正数。两项均在真实 activation 或真实规模前强制重评。

