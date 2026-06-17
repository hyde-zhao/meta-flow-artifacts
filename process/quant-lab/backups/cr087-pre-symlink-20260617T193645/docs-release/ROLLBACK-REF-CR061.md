---
status: "draft-for-cp5-review"
version: "1.0"
change_id: "CR-061"
title: "CR061 Rollback Ref and Preflight Gate"
created_by: "host-orchestrator"
created_at: "2026-06-15T07:29:19+08:00"
execute_allowed: false
---

# CR061 Rollback Ref and Preflight Gate

## 1. 范围和结论

CR061 当前只创建门禁，不创建 commit、不创建新 bundle、不执行 `git reset`、不执行 `git checkout`、不 push/tag/rename remote、不重写历史、不移动文件、不批量改 import。

| 项目 | 当前状态 |
|---|---|
| `pre_cr061_commit` | current HEAD `e7a3a0d`，但工作树含未提交 CR058/059/060/061 文档和 README/USER-MANUAL 改动。 |
| `pre_cr061_git_bundle` | 可引用 CR059 bundle 作为远期冷备，但真实实现前需刷新或风险接受。 |
| `candidate_manifest_hash` | 当前未计算；实现前必须冻结。 |
| dirty worktree policy | 必须有 include/exclude manifest。 |
| `execute_allowed` | false。 |

## 2. Rollback ref checklist

| rollback_ref_id | required_before | evidence_type | evidence_path | status | execute_allowed | 失败处理 |
|---|---|---|---|---|---:|---|
| `pre_cr061_commit` | package metadata update | `commit_hash` | `e7a3a0d` + git status evidence | partial | false | 工作树未清或未冻结 include/exclude 时阻断。 |
| `pre_cr061_git_bundle` | any irreversible bulk action | `git_bundle_verify` 或 risk_acceptance | `/mnt/quant-lab-cold/git-bundles/pre-cr059/local_backtest-pre-cr059-e7a3a0d.bundle` | stale-but-present | false | 真实实现前刷新或用户接受风险。 |
| `candidate_manifest_hash` | any real move / import rewrite | manifest hash | planned | missing | false | 缺失则不允许执行。 |
| `dirty_worktree_include_exclude` | any file edit in CR061 | explicit manifest | planned | missing | false | 缺失则不允许执行。 |
| `failure_manifest_schema` | before execution attempt | schema contract | 本文 §4 | present | false | 仅 schema 存在，不代表可执行。 |

## 3. Dirty worktree policy

| policy_id | policy | 要求 | 失败处理 |
|---|---|---|---|
| DWP-CR061-001 | `clean_preferred` | 真实实现前优先生成 clean worktree 或明确 staged include/exclude。 | BLOCKED。 |
| DWP-CR061-002 | `explicit_include_exclude_manifest_required` | 若工作树不干净，必须列出纳入和排除的用户改动。 | 缺 manifest 则 BLOCKED。 |
| DWP-CR061-003 | `no_user_change_overwrite` | 不得覆盖未纳入计划的用户改动。 | BLOCKED。 |
| DWP-CR061-004 | `uv_lock_generated_only` | `uv.lock` 只能由 uv 命令生成。 | 手改 lock 则 FAIL。 |

## 4. Failure manifest schema

后续如另行授权真实执行，失败记录至少包含：

| 字段 | 必填 | 说明 |
|---|---:|---|
| `failure_id` | yes | `CR061-FAIL-###`。 |
| `path` | yes | repo-relative path。 |
| `action` | yes | `metadata_update` / `compatibility_bridge` / `import_rewrite` / `move`；当前均未授权。 |
| `reason` | yes | 失败原因。 |
| `candidate_id` | yes | 回链 `CR061-CAND-###`。 |
| `rollback_ref` | yes | 对应 rollback ref。 |
| `recovery_action` | yes | 人工恢复或阻断说明。 |
| `owner` | yes | 处理 owner。 |

## 5. Preflight gate

| gate_id | 前置条件 | 当前结果 | 说明 |
|---|---|---|---|
| RBG-CR061-001 | CR061 CP2 / CP3 / CP5 approved | PENDING | 待用户确认。 |
| RBG-CR061-002 | Candidate list present | PASS | `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md` 已创建。 |
| RBG-CR061-003 | Preserve-audit decisions present | PASS | `docs/release/PRESERVE-AUDIT-DECISIONS-CR061.md` 已创建。 |
| RBG-CR061-004 | Commit checkpoint usable | PARTIAL | HEAD 已知但工作树不干净。 |
| RBG-CR061-005 | Git bundle verified or waived | PARTIAL | CR059 bundle present by reference；真实实现前需刷新或 waiver。 |
| RBG-CR061-006 | Candidate manifest hash frozen | BLOCKED | 未计算。 |
| RBG-CR061-007 | Explicit runtime authorization for move / rewrite | BLOCKED | 当前没有。 |

综合结论：`execute_allowed=false`。

## 6. 禁止 Git / 文件操作

| 操作 | 状态 | 说明 |
|---|---|---|
| `git push` / tag / remote rename | not-authorized | 归 CR062。 |
| history rewrite | not-authorized | 禁止。 |
| `git reset --hard` / `git checkout --` | not-authorized | 不得回退用户未明确要求的工作树。 |
| file move / rename / delete | not-authorized | 需要本 CR 内二次授权。 |
| bulk import rewrite | not-authorized | 需要 candidate manifest、preserve-audit allowlist、rollback_ref、full dry-run 和用户授权。 |
