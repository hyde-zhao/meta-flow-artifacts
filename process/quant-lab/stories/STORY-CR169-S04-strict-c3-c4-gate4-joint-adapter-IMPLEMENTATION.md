---
status: completed
version: "1.0"
story_id: "CR169-S04-strict-c3-c4-gate4-joint-adapter"
story_slug: "strict-c3-c4-gate4-joint-adapter"
feature_id: "FEAT-169-02, FEAT-169-03"
implementation_type: "code"
source_story: "process/stories/STORY-CR169-S04-strict-c3-c4-gate4-joint-adapter.md"
source_design_evidence: "process/stories/STORY-CR169-S04-strict-c3-c4-gate4-joint-adapter-LLD.md"
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-15T10:10:00+08:00"
updated_at: "2026-07-15T10:10:00+08:00"
---

# Implementation: CR-169 S04 Strict C3+C4 Gate4 Joint Fixture Adapter

## 1. 实现摘要

新增 CR169-local adapter：typed/self-hash precheck → 13-field exact join → exact 7-key payload → reason/extra guard → public canonical call → non-upgrading postcondition。唯一 PASS 名称为 `gate4_fixture_contract_pass`，四项 readiness/aggregate claims 固定 false。

## 2. 上游设计引用

S04 Story/LLD、FEAT-169-02/03、public `validate_gate4_capacity_impact`、S01-S03 verified evidence、CR168 containment 边界。

## 3. 实现前置检查

S01-S03 CP7 已通过；CP5 approved；public callable/signature 已核验；forbidden owners 无冲突；open items=0。

## 4. 实现对象清单

| 对象 | 目标 | 验证 |
|---|---|---|
| `engine/capacity_liquidity_gate4_projection.py` | Protocol/context/outcome/precheck/call/postcondition | 27 S04 tests |
| `tests/research/test_capacity_liquidity_gate4_projection.py` | header/ref/reason/non-PASS/double/source/claims | pytest |

## 5. 设计契约映射

- 13-field mismatch 13/13：canonical calls=0。
- C4 3 refs typed present：空任一项 N11/calls=0。
- exact 7-key + reason/extra denylist：本地 private guard，不依赖 canonical private helper。
- public DI Protocol：keyword-only `gate4_validator`，production default 为 public callable。
- postcondition：canonical non-PASS 不升级；unexpected malformed PASS REJECTED。

## 6. 单元测试 / Fixture 结果

S04 27 项；S01-S04/CR168/C2/C3 targeted regression 总计 108 项，全 PASS。

## 7. 最小实现切片

Protocol/outcome、correlation/payload precheck、public call/postcondition、source/claim guards 全 done。

## 8. 变更说明

只创建 local adapter 与测试；canonical、CR168 adapter、admission package 变更数=0。

## 9. 平台差异处理

N/A。

## 10. 验证结果

pytest 108 PASS；py_compile/diff-check PASS；forbidden source diff=0。

## 11. 未覆盖项

Canonical 全局 N/A 语义 hardening 和 aggregate integration 留 FU-007；CR-wide fixtures/Stage2 exit 留 S05。

## 12. 风险与回滚

删除 local adapter 即可回滚，不影响 C4 component、CR168 adapter 或 canonical。任何 public signature/semantics 变化必须回 CP3/新 CR。

## 13. 设计缺口反馈

无；评审指出的 canonical false-PASS 风险由 present-path strict allowlist/denylist/postcondition 局部封闭，全局修复仍未越界吞并。

## 14. QA / Review / Doc 后续交接

QA 重点复核 13 mismatch、3 refs、reason/extra、non-PASS 与 malformed PASS；CP8 必须明示 fixture PASS 非 aggregate/admission/real/Stage3。
