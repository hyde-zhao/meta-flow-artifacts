---
story_id: CR101-S04-docs-validation-and-follow-up-gates
change_id: CR-101
title: Docs, validation matrix, and follow-up authorization gates
status: verified
lld_policy: full-lld
wave_id: CR101-W3-DOCS-GATES
depends_on:
  - CR101-S01-target-taxonomy-manifest-contract
  - CR101-S02-package-checker-fixture-fail-closed
  - CR101-S03-runner-adapter-evidence-boundary
owner: host-orchestrator
story_lld: process/stories/CR101-S04-docs-validation-and-follow-up-gates-LLD.md
cp5_check: process/checks/CP5-CR101-S04-docs-validation-and-follow-up-gates-LLD-IMPLEMENTABILITY.md
cp6_check: process/checks/CP6-CR101-S04-docs-validation-and-follow-up-gates-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S04-docs-validation-and-follow-up-gates-VERIFICATION-DONE.md
implementation_evidence: process/stories/CR101-S04-docs-validation-and-follow-up-gates-IMPLEMENTATION.md
---

# CR101-S04 Docs, validation matrix, and follow-up authorization gates

## 用户价值

把 CR101 的离线结论、真实系统不授权边界和后续 gate 清楚留痕，避免后续把离线 READY_WITH_RISK 当作真实 QMT/NAS/MiniQMT ready。

## 范围

- 更新 CR101 docs / validation matrix / CP7 / CP8 预期。
- 记录后续 QMT direct-run real validation、MiniQMT gateway adapter validation、NAS real exchange、order-write gates。
- 保留不授权边界。

## 验收标准

| AC | 描述 |
|---|---|
| S04-AC-01 | docs 明确离线 contract readiness 不等于真实 runtime readiness。 |
| S04-AC-02 | 后续 gate 候选和启动条件明确。 |
| S04-AC-03 | CP7/CP8 允许 READY_WITH_RISK 但必须列 residual risks。 |
| S04-AC-04 | 不新增真实运行脚本，不读取凭据、账户、原始日志。 |

## 文件所有权

| 类型 | 文件 |
|---|---|
| primary | `docs/qmt/*CR101*`、CR101 process checks / checkpoints / context |
| shared | docs/quality 或 docs/release 中 CR101 相关报告 |
| forbidden | 真实 runtime logs、账户数据、凭据、NAS 内容 |

## 测试设计

- 文档 guardrail / grep 检查不授权项。
- CP5 / CP8 human gate decision brief 校验。
- follow-up tracking consistency check。
