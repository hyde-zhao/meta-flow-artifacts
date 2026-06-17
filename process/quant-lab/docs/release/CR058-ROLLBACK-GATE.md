---
status: "implemented-cp6-static"
version: "1.0"
change_id: "CR-058"
title: "CR058 Rollback Ref and Preflight Evidence Gate"
created_by: "meta-dev"
created_at: "2026-06-14T15:40:00+08:00"
source_story: "process/stories/CR058-S04-rollback-ref-and-preflight-evidence-gate.md"
source_lld: "process/stories/CR058-S04-rollback-ref-and-preflight-evidence-gate-LLD.md"
execution_mode: "static-only-no-op"
git_remote_authorized: false
destructive_git_authorized: false
---

# CR058 Rollback Gate

## 1. 范围和结论

本文定义后续任何真实 repo-local move / rewrite 前必须满足的 rollback_ref 和 preflight evidence。当前只写静态 gate，不创建 commit、不创建 bundle、不运行 `git bundle`、不执行 `git reset`、不执行 `git checkout`、不 push/tag/rename remote、不重写历史。

| 项目 | 当前状态 |
|---|---|
| `pre_cr058_commit` | planned / missing；本轮不创建。 |
| `pre_cr058_git_bundle` | planned / missing；本轮不创建。 |
| `pre_reference_rewrite_manifest` | planned / missing；本轮仅定义合同。 |
| `candidate_manifest_hash` | planned / missing；本轮不计算。 |
| dirty worktree policy | planned；后续真实执行前必须检查。 |
| `execute_allowed` | false。 |

## 2. Rollback ref checklist

| rollback_ref_id | required_before | evidence_type | evidence_path | status | execute_allowed | 失败处理 |
|---|---|---|---|---|---:|---|
| `pre_cr058_commit` | `any_real_move_or_rewrite` | `commit_hash` | planned | missing | false | 缺失则阻断真实 move / rewrite。 |
| `pre_cr058_git_bundle` | `any_irreversible_bulk_action` | `git_bundle_verify` 或 `risk_acceptance` | planned | missing | false | 缺失且无用户 risk_acceptance 时阻断。 |
| `pre_reference_rewrite_manifest` | `bulk_reference_rewrite` | `manifest_hash` | planned | missing | false | 缺失则不允许 reference rewrite。 |
| `candidate_manifest_hash` | `any_real_move_or_rewrite` | `manifest_hash` | planned | missing | false | 缺失则候选范围不可冻结。 |
| `failure_manifest_schema` | `before_execution_attempt` | `schema_contract` | 本文 §4 | present | false | 仅 schema 存在，不代表可执行。 |

## 3. Dirty worktree policy

| policy_id | policy | 要求 | 失败处理 |
|---|---|---|---|
| DWP-CR058-001 | `clean_required` | 后续真实执行前必须有可审查的 clean worktree 证据。 | BLOCKED。 |
| DWP-CR058-002 | `explicit_include_exclude_manifest_required` | 若工作树不干净，必须列出纳入和排除的用户改动。 | 缺 manifest 则 BLOCKED。 |
| DWP-CR058-003 | `no_user_change_overwrite` | 不得覆盖未纳入计划的用户改动。 | BLOCKED，交回 host-orchestrator。 |

本轮未运行 dirty worktree 变更操作；只允许后续 QA 用 `git diff --check` 进行格式验证。

## 4. Failure manifest schema

后续如另行授权真实执行，失败记录至少包含：

| 字段 | 必填 | 说明 |
|---|---:|---|
| `failure_id` | yes | `CR058-FAIL-###`。 |
| `path` | yes | 失败对象 repo-relative path。 |
| `action` | yes | `move` / `rewrite` / `rename` / `delete`；当前均未授权。 |
| `reason` | yes | 失败原因。 |
| `candidate_id` | yes | 回链 `CR058-CAND-###`。 |
| `rollback_ref` | yes | 对应 rollback ref。 |
| `recovery_action` | yes | 人工恢复或阻断说明。 |
| `owner` | yes | 处理 owner。 |

## 5. Preflight gate

| gate_id | 前置条件 | 当前结果 | 说明 |
|---|---|---|---|
| RBG-CR058-001 | CR058 CP5 approved | PASS | `process/checkpoints/CP5-CR058-RELAYOUT-GATE-BATCH-A-LLD-BATCH.md` approved。 |
| RBG-CR058-002 | Candidate list present | PASS | `docs/release/CR058-CANDIDATE-LIST.md` exists after CP6 implementation。 |
| RBG-CR058-003 | Preserve-audit allowlist present | PASS | `docs/release/CR058-PRESERVE-AUDIT-ALLOWLIST.md` exists after CP6 implementation。 |
| RBG-CR058-004 | Commit checkpoint present | BLOCKED | 本轮不创建 commit。 |
| RBG-CR058-005 | Git bundle verified or waived | BLOCKED | 本轮不创建 bundle，也无 risk_acceptance waiver。 |
| RBG-CR058-006 | Candidate manifest hash frozen | BLOCKED | 本轮不计算 hash。 |
| RBG-CR058-007 | Explicit runtime authorization | BLOCKED | 用户未授权真实 move / rewrite。 |

综合结论：`execute_allowed=false`。

## 6. 禁止 Git / 文件操作

| 操作 | 状态 | 说明 |
|---|---|---|
| `git push` / tag / remote rename | not-authorized | 需要独立 git remote gate。 |
| history rewrite | not-authorized | 禁止。 |
| `git reset --hard` / `git checkout --` | not-authorized | 不得回退用户未明确要求的工作树。 |
| file move / rename / delete | not-authorized | 需要后续明确 runtime_authorization。 |
| bulk path rewrite | not-authorized | 需要 candidate manifest、preserve-audit allowlist、rollback_ref 和用户授权。 |

## 7. 后续验证入口

| 验证对象 | 入口 | 预期 |
|---|---|---|
| rollback ref fail-closed | 本文 §2 / §5 | 缺 commit / bundle / manifest 时 `execute_allowed=false`。 |
| destructive Git 禁止 | 本文 §6 与 CP6 check | 只能作为禁止项出现。 |
| no-op 边界 | `docs/release/CR058-NO-OP-GUARDRAILS.md` | 不授权项完整。 |
