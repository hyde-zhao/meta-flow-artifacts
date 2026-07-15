---
status: completed
version: "1.0"
story_id: "CR169-S05-fixture-claim-stage2-exit-verification"
story_slug: "fixture-claim-stage2-exit-verification"
feature_id: "FEAT-169-01, FEAT-169-02, FEAT-169-03, FEAT-169-04"
implementation_type: "tests-fixtures-checker"
source_story: "process/stories/STORY-CR169-S05-fixture-claim-stage2-exit-verification.md"
source_design_evidence: "process/stories/STORY-CR169-S05-fixture-claim-stage2-exit-verification-LLD.md"
created_by: "host-orchestrator-inline-meta-dev"
created_at: "2026-07-15T09:58:00+08:00"
updated_at: "2026-07-15T09:58:00+08:00"
---

# Implementation: CR-169 S05 Fixture、Claim 与 Stage2 Exit 验证

## 1. 实现摘要

新增 2/2 fixture、CR-169 精确 QAC 与 claim regression 测试，以及只消费显式仓库内 evidence refs 的 Stage2 七项合同退出核验器。核验器不扫描外部数据，不修补历史合同；总 PASS 当且仅当七项全部 PASS，任何结果下 `stage3_entry_ready=false`。

## 2. 上游设计引用

S05 Story/LLD、FEAT-169-01..04、S01-S04 CP7 结果、ADR-005/006、CP5 批量批准范围。

## 3. 实现前置检查

S01-S04 已分别通过 CP6/CP7；CP5 已批准；S05 work packet 完备性检查通过；禁止修改 owner 无冲突；open item=0。

## 4. 实现对象清单

| 对象 | 目标 | 验证 |
|---|---|---|
| `tests/fixtures/capacity_liquidity/daily_multifactor_v1.json` | 日频多因子 synthetic C4 happy path、能力注册 N/A 与 claim ceiling | S05 QAC |
| `tests/fixtures/capacity_liquidity/multi_strategy_compatibility_v1.json` | daily/ML 同计算语义、不同 attachment identity | S05 QAC |
| `tests/research/test_capacity_liquidity_cr169_qac.py` | 9/17/15/12、2/2、10→1、Stage2 schema/routing、安全边界 | 10 tests |
| `tests/research/test_capacity_liquidity_claim_regression.py` | CR168 absent、CR155 与 real/aggregate/Stage3 false | 3 tests |
| `tests/PROVENANCE.yaml` | 登记 CR-169 新增的 6 个测试文件、Story 与 CR 来源 | taxonomy test |
| `scripts/check_stage2_exit_contracts.py` | 7-item result builder/CLI、路径约束、失败路由 | QAC + py_compile |

## 5. 设计契约映射

- fixture family 精确 2 个，不包含 source locator、credential 或真实数据。
- 产品基线 ID 通过仓库正式文档集合校验：REQ=9、scenario=17、QAC=15。
- `C4_REASON_CODES` 精确 12 且唯一；10 次 producer 运行只产生 1 个 component hash。
- capability registry 缺失使用 `not_applicable_with_reason` 和既有 Feature refs，不伪造运行时 registry。
- Stage2 checker 固定七项、精确路由；前六项失败指向 CR-157 owner/治理 CR，第七项失败指向 CR-169 rework。

## 6. 单元测试 / Fixture 结果

S05 定向 13 项全部通过；S01-S05、CR168、CR155、C2/C3 联合回归 115 项全部通过。

## 7. 最小实现切片

两族 fixture、QAC、claim regression、Stage2 result builder/CLI 全部完成。正式 Stage2 result 只在 CP8 装配真实 evidence refs 后生成，CP6 未预生成伪 PASS。

## 8. 变更说明

只新增 S05 所有权内测试、fixture 和 checker。`engine/cross_strategy_reliability_gates.py`、`engine/economic_cost_gate4_projection.py`、`engine/strategy_admission_package.py` 修改数均为 0。

## 9. 平台差异处理

N/A；Python 命令统一经 `uv run --python 3.11`。

## 10. 验证结果

定向 pytest 13 PASS；CR-wide/regression pytest 115 PASS；首轮 full suite 的 provenance 缺口已在 CP7 内整改；`py_compile`、`git diff --check` 与 forbidden diff 均 PASS。

## 11. 未覆盖项

Repository full suite、正式七项历史 evidence 装配和 CP8 发布就绪验证留 CP7/CP8；真实 ADV/liquidity、alpha-decay、FU-007 aggregate/canonical hardening 均不在范围。

## 12. 风险与回滚

删除新增 fixture/tests/checker 即可回滚，不影响 S01-S04 production contract。历史合同缺口不得通过修改 CR-169 fixture 规避。

## 13. 设计缺口反馈

无新设计缺口；评审要求的 capability registry N/A 归属和 Stage2 失败补救边界均已有可执行测试。

## 14. QA / Review / Doc 后续交接

QA 必须复核 2/2、9/17/15/12、10→1、joint/absent、CR155、source diff、full suite 与正式 7/7；CP8 必须披露 inline verifier 风险，并明确 Stage2 complete 不等于 Stage3 ready。
