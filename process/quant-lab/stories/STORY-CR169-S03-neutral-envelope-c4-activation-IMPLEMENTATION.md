---
status: completed
version: "1.0"
story_id: "CR169-S03-neutral-envelope-c4-activation"
story_slug: "neutral-envelope-c4-activation"
feature_id: "FEAT-169-01, FEAT-166-01"
implementation_type: "mixed"
source_story: "process/stories/STORY-CR169-S03-neutral-envelope-c4-activation.md"
source_design_evidence: "process/stories/STORY-CR169-S03-neutral-envelope-c4-activation-LLD.md"
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-15T09:51:00+08:00"
updated_at: "2026-07-15T09:51:00+08:00"
---

# Implementation: CR-169 S03 Neutral Envelope C4 激活

## 1. 实现摘要

在既有 `COMPONENT_CATALOG` 增量激活 `capacity_liquidity@v1`；未修改 neutral envelope class、builder、validator、serializer 或 hash domain。新增 daily/ML attachment、unavailable、tamper、C1-C3 compatibility 和 no-parallel-API tests。

## 2. 上游设计引用

S03 Story/LLD、CR166 neutral envelope contract、S01/S02 verified C4 evidence、ADR-001/002。

## 3. 实现前置检查

S01/S02 CP7 PASS_WITH_RISK；CP5 approved；catalog owner/file conflict 均满足；open items=0。

## 4. 实现对象清单

| 对象 | 动作 | 验证 |
|---|---|---|
| `engine/strategy_evidence.py` | 增加一个 active tuple | catalog/regression |
| `tests/research/test_capacity_liquidity_envelope_compatibility.py` | 新建 | 5 tests + C2/C3 regressions |

## 5. 设计契约映射

- active C4 descriptor=1：`component_catalog_status(...,"v1")`。
- reserved 仍为非 active compatibility placeholder；unknown v2 保持 UNKNOWN。
- component identity 与 envelope identity 分域：daily/ML 1 component hash / 2 envelope hash。
- parallel envelope/registry/discovery=0：public API negative assertions。

## 6. 单元测试 / Fixture 结果

S01/S02/S03 + C2/C3 envelope/QAC targeted regression 共 66 passed。

## 7. 最小实现切片

Catalog activation、compatibility tests、targeted regressions 均 done。

## 8. 变更说明

仅两文件变更；C1/C2/C3 schema/hash 与 canonical Gate4/CR168 adapter/admission package 均未修改。

## 9. 平台差异处理

N/A。

## 10. 验证结果

pytest 66 PASS、py_compile PASS、diff-check PASS、forbidden diff=0。

## 11. 未覆盖项

13-field joint correlation 与 Gate4 composition 归 S04；CR-wide fixture/claims 归 S05。

## 12. 风险与回滚

若兼容失败，只撤回 active v1 tuple，保留 reserved 占位与 S01/S02 unattached component；无迁移或 store 写入。

## 13. 设计缺口反馈

无。

## 14. QA / Review / Doc 后续交接

QA 重跑 active/reserved/unknown、daily/ML hash、tamper 和 C2/C3；CP8 文案不得把 envelope PRESENT 等同 aggregate admission 或真实 capacity readiness。
