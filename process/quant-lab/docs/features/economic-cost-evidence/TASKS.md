---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-01"
---

# FEAT-168-01 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 初始 S01/S02/S03 执行任务。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：任务显式要求 N01..N10、exact basis/net return 和 producer issue short-circuit。 |

| TASK-ID | Story | 顺序 | 动作 | 输出文件 | 完成准则 |
|---|---|---:|---|---|---|
| EC-K01 | S01 | 1 | 创建 C3 immutable value/schema/domain constants | `engine/economic_cost_evidence.py` | component/schema=1/1；family 1 与 semantic projection 分离。 |
| EC-K02 | S01 | 2 | 创建 9-family normalizer/validator/reason table | 同上 | N01..N10 的精确 code、稳定排序与 availability effect；false PASS=0。 |
| EC-K03 | S01 | 3 | 创建 component semantic hash/self-validation | 同上 | 10→1；identity excluded；tamper blocked。 |
| EC-K04 | S02 | 4 | 创建 Decimal itemized/square-root calculator | `engine/economic_cost_calculator.py` | 五个 exact basis、proxy domain、precision/rounding/minor unit/net return 合同精确。 |
| EC-K05 | S02 | 5 | 组合 producer/availability/reconciliation | `engine/economic_cost_evidence.py` | normalize→validate→issues short-circuit→calculate→produce；issue path calculator=0；no I/O。 |
| EC-K06 | S01/S02 | 6 | 创建 contract/producer tests | 两个 economic-cost test 文件 | TEST-PLAN T01..T10 可执行。 |
| EC-K07 | S03/S05 | 7 | 提供 neutral attach 与跨策略验证所需 typed API | public exports only | 不把 package identity 写入 component。 |

阻塞项：无。任何 schema/hash/rounding 变动先回 CP3/design clarification；CP5 前禁止执行任务。
