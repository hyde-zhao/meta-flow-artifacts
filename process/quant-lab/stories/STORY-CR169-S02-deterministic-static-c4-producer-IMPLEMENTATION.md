---
status: completed
version: "1.0"
story_id: "CR169-S02-deterministic-static-c4-producer"
story_slug: "deterministic-static-c4-producer"
feature_id: "FEAT-169-01"
implementation_type: "code"
source_story: "process/stories/STORY-CR169-S02-deterministic-static-c4-producer.md"
source_design_evidence: "process/stories/STORY-CR169-S02-deterministic-static-c4-producer-LLD.md"
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-15T09:40:00+08:00"
updated_at: "2026-07-15T09:40:00+08:00"
---

# Implementation: CR-169 S02 确定性 Static C4 Producer

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 实现 `static_adv_cap_v1` pure Decimal calculator 与唯一 C4 producer。 |
| 行为变化 | validation-clean 输入生成 3/3 typed refs 和 `capacity_liquidity@v1`；issues 在 calculator 前短路。 |
| 范围边界 | fixture/static-only；不接真实数据、不激活 envelope、不调用 Gate4、不实现 alpha-decay。 |
| CP6 证据 | `process/checks/CP6-CR169-S02-IMPLEMENTATION-DONE.result.json` |

## 2. 上游设计引用

S01 verified contract、S02 LLD、FEAT-169-01 DESIGN/TEST-PLAN，以及 HLD `static_adv_cap_v1`/claim ceiling。

## 3. 实现前置检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| S01 CP7 | PASS | typed validation/header/hash 可消费。 |
| LLD/open items | PASS | CP5 approved；open_items=0。 |
| Wave/file owner | PASS | W2 串行，shared evidence module merge owner=S02。 |
| 验证方式 | PASS | numeric golden、cap/rounding、hash、claim tests。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 验证 |
|---|---|---|---|
| `engine/capacity_liquidity_calculator.py` | code | precision=28、fixed formula、HALF_EVEN。 | unit/py_compile |
| `engine/capacity_liquidity_evidence.py` | code | producer/ref payload/evidence/self-validation。 | contract/integration |
| `tests/research/test_capacity_liquidity_producer.py` | test | formulas、3 refs、cap、rounding、10→1、claims。 | pytest |

## 5. 设计契约映射

| 要求 | 实现位置 | 验证 |
|---|---|---|
| exact formula / precision=28 | calculator | numeric golden |
| ratio=cap PASS、ratio>cap BLOCKED | producer post-calculation guard | cap edge |
| issues 非空 calculator=0 | producer entry | short-circuit test |
| 3/3 refs content-addressed | `CapacityLiquidityRefPayloadV1` | ref triplet test |
| no-real 与 alpha=0 | EvidenceV1 defaults/self-validation | claim ceiling test |

## 6. 单元测试 / Fixture 计划与结果

10 个 S02 用例与 27 个 S01 回归用例全部通过；覆盖 formula、cap 域、HALF_EVEN、tamper、currency 名称陷阱和 no-real claims。

## 7. 最小实现切片

| Slice | 内容 | 状态 |
|---|---|---|
| S02-01 | calculator/breakdown | done |
| S02-02 | producer/ref/evidence/self-validation | done |
| S02-03 | targeted tests + S01 regression | done |

## 8. 变更说明

- 创建 calculator 与 producer tests。
- 增量扩展 S01 evidence module；保留既有 S01 API。
- forbidden modules 变更数为 0。

## 9. 平台差异处理

N/A；纯 Python repository-local 逻辑。

## 10. 验证结果

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 pytest -q tests/research/test_capacity_liquidity_contracts.py tests/research/test_capacity_liquidity_producer.py` | PASS（37） |
| py_compile | PASS |
| git diff --check | PASS |
| forbidden module diff | PASS（0） |

## 11. 未覆盖项

Envelope attachment、joint Gate4 adapter 和 CR-wide fixture/Stage2 退出分别由 S03/S04/S05 承担。

## 12. 风险与回滚

synthetic proxy 只能用于 fixture；`real_adv_available=false`、`real_liquidity_available=false`、`capacity_ready=false` 固化。回滚时删除 S02 calculator/producer 增量并保留 S01 typed-unavailable contract。

## 13. 设计缺口反馈

无设计偏差或开放问题。

## 14. QA / Review / Doc 后续交接

QA 独立复跑 formula/cap/rounding/hash/claim；Review 核对无 I/O 和 content-addressed ref；CP8 明示 `capacity_dollars_ref` 不代表 USD 或真实 capacity。
