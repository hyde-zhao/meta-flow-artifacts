---
status: completed
version: "1.0"
story_id: "CR169-S01-capacity-liquidity-contract-validation"
story_slug: "capacity-liquidity-contract-validation"
feature_id: "FEAT-169-01, FEAT-169-02"
implementation_type: "mixed"
source_story: "process/stories/STORY-CR169-S01-capacity-liquidity-contract-validation.md"
source_design_evidence: "process/stories/STORY-CR169-S01-capacity-liquidity-contract-validation-LLD.md"
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-15T09:26:00+08:00"
updated_at: "2026-07-15T09:26:00+08:00"
---

# Implementation: CR-169 S01 C4 合同、关联头与输入校验

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 建立 `capacity_liquidity@v1` typed input、13 字段 correlation header、N01..N12 稳定原因码与 subject-neutral semantic hash。 |
| 行为变化 | 新增 repository-local 纯值合同；合法输入可形成 S02 所需 typed 四元，非法输入在计算前 fail-closed。 |
| 范围边界 | 未实现 calculator/producer、envelope 激活、Gate4 调用、真实数据或 alpha-decay。 |
| CP6 证据 | `process/checks/CP6-CR169-S01-IMPLEMENTATION-DONE.result.json` |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story / LLD | `process/stories/STORY-CR169-S01-capacity-liquidity-contract-validation.md` / `-LLD.md` | 范围、接口、13 字段、N01..N12、hash 分域。 |
| HLD / ADR | `process/docs/design/HLD-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md` / `ARCHITECTURE-DECISION-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md` | static-only、exact header、schema v1。 |
| Feature | `process/docs/features/capacity-liquidity-evidence/DESIGN.md`、`process/docs/features/c3-c4-correlation-boundary/DESIGN.md` | typed values、join 与 no-real 边界。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| 上游 Feature 设计存在 | PASS | FEAT-169-01/02 完整。 |
| Story 范围、问题、验证方式明确 | PASS | CP5 已批准，open_items=0。 |
| 当前 Wave / dev_gate 满足 | PASS | S01 `dev-ready`、无依赖。 |
| 文件所有权无冲突 | PASS | 只创建 S01 primary files。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 必须 | 验证方式 |
|---|---|---|---|---|
| `engine/capacity_liquidity_evidence.py` | code / template-schema | typed values、normalizer、validator、header/hash。 | yes | unit/contract/py_compile |
| `tests/research/test_capacity_liquidity_contracts.py` | guardrail-test | 13-field、reason、hash、tamper、no-I/O。 | yes | pytest |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 验证 |
|---|---|---|---|
| header 13/13 exact fields | LLD §8.1 | `C3C4CorrelationHeaderV1` / `CORRELATION_HEADER_FIELDS` | 13 个逐字段 mismatch 测试 |
| N01..N12 stable ordering | LLD §8.2 | `C4_REASON_CODES` / `_ISSUE_ORDER` | 枚举数量与 fail-closed matrix |
| identity 不进入 component hash | CP3 DQ-HEADER | `semantic_projection()` | daily/ML identity split 10→1 |
| issue path 先于 calculator | LLD §7 | `CapacityLiquidityValidationResult` | S02 producer 继续消费；S01 calculator=0 |

## 6. 单元测试 / Fixture 结果

| 测试对象 | 类型 | 期望 | 状态 |
|---|---|---|---|
| valid typed contract | unit | issues=0，header=13 | passed |
| N01/N02/N03/N04/N05/N06/N07/N09/N10 | negative | exact reason + availability | passed |
| 13 个 header mismatch | contract | 全部 BLOCKED/N08 | passed |
| identity/hash/tamper/opaque refs | security/fixture | 10→1、tamper blocked、无解引用 | passed |

## 7. 最小实现切片

| Slice ID | 改动对象 | 局部验证 | 状态 |
|---|---|---|---|
| IMPL-S01-01 | constants/value types | import/py_compile | done |
| IMPL-S01-02 | normalize/validate/hash | contract tests | done |
| IMPL-S01-03 | test matrix | 27 passed | done |

## 8. 变更说明

| 文件 | 动作 | 说明 |
|---|---|---|
| `engine/capacity_liquidity_evidence.py` | create | C4 v1 public contract 与 pure validation。 |
| `tests/research/test_capacity_liquidity_contracts.py` | create | 27 项静态/合同测试。 |

## 9. 平台差异处理

N/A；本 Story 是 Python repository-local pure value contract，不涉及 Agent、Skill 或安装器。

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/research/test_capacity_liquidity_contracts.py` | PASS | 27 passed |
| `uv run --python 3.11 python -m py_compile ...` | PASS | exit 0 |
| `git diff --check -- engine/... tests/...` | PASS | exit 0 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| N11/N12 runtime trigger | 属 S04 joint adapter | S04 实现与测试；枚举已由 S01 冻结。 |
| component producer/envelope | 属 S02/S03 | 串行后续 Story。 |

## 12. 风险与回滚

| Risk ID | 风险 | 缓解 / 回滚 |
|---|---|---|
| R-CR169-PROXY-VALIDITY | synthetic 输入被误读为真实容量 | no-real claims 纳入 semantic body；回滚为 reserved/typed-unavailable。 |
| R-CR169-STAGE2-OVERCLAIM | 合同完成被误读为 Stage3 ready | 本 Story 不声明 Stage2 complete；CP8 另做 7/7。 |

## 13. 设计缺口反馈

无阻断设计缺口。实现中明确：单输入 identity 缺失归 N01；N08 只由显式 C3/C4 join 产生，符合 reason owner 分层。

## 14. QA / Review / Doc 后续交接

- QA：重点独立验证 13 个 mismatch、10→1 hash、float/NaN 拒绝和 no-real claims。
- Review：确认无外部 I/O、无 forbidden module 改动、identity 不进入 semantic projection。
- Doc：本 Story 无用户文档变更；Feature 语义未改变。
