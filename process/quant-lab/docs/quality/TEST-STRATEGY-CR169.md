---
title: "CR-169 测试策略"
status: "cp7-complete"
version: "1.0"
cr_id: "CR-169"
validation_mode: "static-fixture-repository"
created_at: "2026-07-15T10:12:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-169 测试策略

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline meta-qa | 冻结 C4 fixture foundation 的风险驱动测试层次、精确覆盖值、失败归因与 CP8 退出条件。 |

## 1. 目标与方法

目标是证明 `capacity_liquidity@v1` 在 fixture/static-only 边界内具有确定、可审计、fail-closed 的输入、计算、身份、envelope 和 Gate4 consumer 合同。策略参考 ISTQB 风险驱动测试与 ISO 25010 的功能正确性、可靠性、安全性、兼容性和可维护性维度；该集合不是穷尽框架。

## 2. 验证层次

| 层次 | 验证对象 | 精确退出条件 |
|---|---|---|
| contract/unit | 13-field header、12 reason、normalization/hash | P0 12/12；header mismatch 13/13 blocked |
| calculation | Decimal static ADV/capacity/headroom | ratio/cap/headroom 精确；10→1 hash |
| envelope compatibility | active catalog、subject-neutral component、subject-bound envelope | active schema=1；parallel envelope/registry=0 |
| security integration | strict C3+C4 7-key adapter、reason denylist、public canonical call、postcondition | valid fixture PASS=1；false promotion=0 |
| regression | CR168 C3-only absent、CR155、C1-C3 | absent regression=1；CR155 promotion=0 |
| product/QAC | 2 fixtures、9 requirements、17 scenarios、15 QAC | 2/2、9/9、17/17、15/15 |
| repository/governance | design surface、provenance、artifact hygiene、full suite | 产品路径 failure=0；工作区敏感失败必须精确归因并在提交后归零 |
| stage transition | Stage2 七项合同 | 7/7 PASS；`stage3_entry_ready=false` |

## 3. 失败策略

任何数值、单位、关联、hash、typed ref、reason escape 或授权问题必须 fail-closed。前六项 Stage2 历史合同失败路由 `CR-157-owner-or-new-governance-CR`；第七项 C4/evidence-index 失败路由 `CR-169-NEEDS_REWORK`。repository full suite 中无法证明来源的失败阻断 CP7。

## 4. 授权与环境

只允许仓库内 fixture、源码、测试和 process evidence 读取；Python 统一通过 `uv`。真实 ADV/liquidity、provider/lake/NAS/credential、runtime/QMT/trading、publish/deploy/tag/release、Git remote write 均不执行。

## 5. 风险接受

Inline verifier independence 与未提交工作区 artifact-hygiene 仅可形成 `PASS_WITH_RISK`。CP8 批准后若获得本地提交授权，必须重跑 full suite；未归零不得关闭 CR 或声称 repository suite 全绿。
