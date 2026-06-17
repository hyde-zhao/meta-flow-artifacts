---
status: "pending-cp5-approval"
version: "1.0"
change_id: "CR-060"
title: "Rollback Ref for CR060"
created_by: "host-orchestrator"
created_at: "2026-06-15T06:43:41+08:00"
---

# CR060 Rollback Ref

## 1. 当前回滚证据

| rollback_ref_id | evidence_type | evidence_path | status | 说明 |
|---|---|---|---|---|
| `pre_cr060_source_commit` | commit_hash | `e7a3a0d7e66b3972b95c047f017790d9853625d0` | present | manifest 创建时 HEAD 未变。 |
| `pre_cr060_git_bundle` | git_bundle | `/mnt/quant-lab-cold/git-bundles/pre-cr059/local_backtest-pre-cr059-e7a3a0d.bundle` | present-via-CR059 | CR059 已 verify PASS；若 HEAD 变化需刷新。 |
| `pre_cr060_cold_backup` | cold_backup | `/mnt/quant-lab-cold/backups/pre-migration/cr059/repo-data-snapshot-20260614` | present-via-CR059 | docs-only 改写可复用；数据迁移前必须刷新。 |
| `cr060_execution_manifest` | manifest | `docs/release/MIGRATION-EXECUTION-MANIFEST-CR060.yaml` | present | 执行清单已冻结，CP5 未批前不可执行。 |

## 2. 回滚动作

| 场景 | 回滚方式 | 限制 |
|---|---|---|
| README / USER-MANUAL 改写后发现语义错误 | 使用 Git diff 反向补丁修订 touched lines | 不使用 `git reset --hard`。 |
| 工作树混入未授权改动 | 停止执行，记录 failure manifest，由人工决定 include/exclude | 不覆盖用户改动。 |
| 需要恢复到 pre-CR060 内容 | 从 CR059 cold backup 或 Git bundle 对照恢复指定文件 | 只恢复明确列出的文件。 |
| 发现需要包名 / import 改动 | 停止 CR060，转 CR061 | 不在本 CR 扩大范围。 |

## 3. 失败记录 schema

| 字段 | 必填 | 说明 |
|---|---:|---|
| `failure_id` | yes | `CR060-FAIL-###` |
| `action_id` | yes | 回链 manifest 中的 action ID |
| `path` | yes | repo-relative path |
| `reason` | yes | 失败原因 |
| `rollback_ref` | yes | 使用的回滚证据 |
| `recovery_action` | yes | 修复、回滚或阻断方式 |
| `owner` | yes | 处理 owner |

## 4. 当前结论

`execute_allowed_now=false`。用户通过 CP5 后，仅允许执行 `CR060-ACT-001` 和 `CR060-ACT-002`。
