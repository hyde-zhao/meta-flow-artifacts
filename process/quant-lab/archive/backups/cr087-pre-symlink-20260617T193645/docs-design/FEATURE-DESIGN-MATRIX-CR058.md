---
status: "draft-for-review"
version: "0.1"
change_id: "CR-058"
created_by: "host-orchestrator"
created_at: "2026-06-14T13:59:00+08:00"
---

# Feature Design Matrix CR058

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-14 | host-orchestrator | 新增 CR058 Feature 设计矩阵草案。 |

## Feature Matrix

| Feature | 范围 | 设计要求 | 设计证据 | Story | lld_policy |
|---|---|---|---|---|---|
| FEAT-CR058-01 Intake and Conflict Gate | FU 转正式 CR、CR046 paused non-overlap、CR053 closed 输入 | required | 本 HLD、CR058 formal CR | CR058-S01 | technical-note |
| FEAT-CR058-02 Candidate List Gate | repo-local候选清单、owner、classification、risk、verification | required | 本 HLD、CR053 inventory | CR058-S02 | full-lld |
| FEAT-CR058-03 Preserve-audit and Sensitive Filter Gate | 历史审计保护、敏感输入排除 | required | 本 HLD、PATH-REFERENCES-CR053 | CR058-S03 | full-lld |
| FEAT-CR058-04 Rollback Ref Gate | commit、bundle、manifest 前置 | required | 本 HLD、MIGRATION-PLAN-CR053 | CR058-S04 | full-lld |
| FEAT-CR058-05 No-op Authorization Boundary | dry-run 设计、真实动作不授权、后续 CP6/CP8 边界 | required | 本 HLD、ADR | CR058-S05 | technical-note |

## Story 下游消费契约

| Story | required_level | feature_design_refs | dev_gate |
|---|---|---|---|
| `CR058-S01-intake-conflict-and-scope-freeze` | technical-note | FEAT-CR058-01 | CP2 approved；不恢复 CR046。 |
| `CR058-S02-candidate-list-and-owner-classification-gate` | full-lld | FEAT-CR058-02 | candidate list schema approved；不执行 move。 |
| `CR058-S03-preserve-audit-allowlist-and-sensitive-filter-gate` | full-lld | FEAT-CR058-03 | preserve-audit allowlist approved；敏感输入 excluded。 |
| `CR058-S04-rollback-ref-and-preflight-evidence-gate` | full-lld | FEAT-CR058-04 | rollback_ref checklist approved；缺失 fail-closed。 |
| `CR058-S05-mechanical-migration-dry-run-and-authorization-boundary` | technical-note | FEAT-CR058-05 | no-op boundary approved；真实执行另行授权。 |
