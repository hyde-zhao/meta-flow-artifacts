---
status: draft
version: "1.0"
source_cr: "CR-093"
created_at: "2026-06-18T17:55:00+08:00"
owner: "host-orchestrator"
---

# FEATURE-DESIGN-MATRIX — CR093 Ledger Hygiene

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-18 | host-orchestrator | 初始矩阵，判定 CR093 ledger hygiene 需要 Feature 级设计和 full-lld。 |

## Feature 判定

| Feature ID | Feature | 判定 | 触发原因 | 关联 Story | Feature 设计证据 | Story lld_policy |
|---|---|---|---|---|---|---|
| F-CR093-01 | CR tracking current-state semantic checker | required | 修改状态机检查器、历史账本语义、错误分类和门禁输出；影响跨 CR tracking 合同 | CR093-LEDGER-HYGIENE-CONSISTENCY | `docs/features/CR093-ledger-hygiene/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld |

## Waived / N/A

| 对象 | 判定 | 理由 | 重访条件 |
|---|---|---|---|
| 全局 Meta Flow 状态机重写 | waived | CR093 只治理 CR019 / CR025 当前阻断项，不重写全局状态机 | checker 规则继续膨胀或 warning-only 项成为阻断 |
| NAS / runtime / account evidence | n/a | CP2 / CP3 均明确禁止外部动作 | 用户另起 runtime / NAS gate |

## 下游消费契约

| Story | 输入 | 输出 | 文件 owner | merge order |
|---|---|---|---|---|
| CR093-LEDGER-HYGIENE-CONSISTENCY | HLD、CR093 CR、CR019 tracking、checker 当前实现 | checker 语义修正、最小台账规范化、测试和 CP6/CP7 证据 | `scripts/check_cr_tracking_consistency.py`、`process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md`、`tests/test_cr093_cr_tracking_consistency.py` | 1. checker tests；2. checker parser；3. CR019 最小规范化；4. STATE / CR-INDEX 状态同步 |
