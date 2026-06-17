---
project_id: "meta-flow"
release_scope: "meta-flow-artifact-routing-and-eval-governance"
created_at: "2026-06-17T13:49:25+08:00"
---

# Rollback

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-11 | host-orchestrator | 新增 workflow eval governance 回滚方案 |
| 1.1 | 2026-06-17 | host-orchestrator | 增加 artifact routing 与 advanced eval runner 回滚说明 |

## 回滚范围

如需回滚 workflow eval governance，撤销以下对象：

- `meta_flow/evals/*`
- `meta_flow/cli.py` 中 `eval` 子命令
- `evals/contracts/*`
- `evals/fixtures/generated-workflow-basic/*`
- `evals/fixtures/generated-workflow-advanced/*`
- `evals/adapters/ADAPTERS.md`
- `evals/ci/CI-POLICY.md`
- workflow eval governance 相关 docs / rules / template 修改

如需回滚 artifact routing，必须先人工确认 artifact repo 备份可读，再执行：

| 对象 | 回滚方式 | 验证 |
|---|---|---|
| `process/` symlink | 删除 symlink，恢复 `/home/hyde/projects/.meta-flow-migration-backups/process.local-backup-CR026-*` 到源码仓库 | `meta-flow status` 可读且 CR tracking PASS |
| `docs/` symlink | 删除 symlink，恢复 `/home/hyde/projects/.meta-flow-migration-backups/docs.local-backup-CR027-*` 到源码仓库 | release/docs/design 关键文件可读 |
| artifact repo tracking | 保留 artifact repo 历史，不删除远端 | `git status` clean |

## 回滚验证

- `meta-flow workspace check` 对目标路由状态给出明确 PASS 或 BLOCKED。
- `meta-flow check cr-tracking --project-root .` 仍 PASS。
- guardrail 无新增错误。
- 若回滚 eval 子命令，`meta-flow --help` 不再展示 eval 子命令。

## 不可静默回滚项

- 不得删除 `/home/hyde/projects/meta-flow-artifacts` 或其远端历史。
- 不得在用户未确认 artifact root 时重建空 `process/STATE.md`。
- 不得把 CP8 approve 解释为真实发布、远端删除或生产写入授权。
