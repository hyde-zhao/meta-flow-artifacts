---
check_id: "CR120-DIRECTORY-ROUTING-MERGE-AUDIT"
status: "PASS_WITH_FINDINGS"
created_at: "2026-06-23T08:58:36+08:00"
created_by: "host-orchestrator"
source_request: "user CP8 approve follow-up analysis"
---

# CR120 Directory Routing Merge Audit

## 结论摘要

`docs/design` 和 `docs/features` 当前仍是指向 artifact repo 的 symlink，没有被 2026-06-22 merge 还原为普通目录。混乱点不是这两个目录本体被覆盖，而是整改后应忽略或外置的路径仍出现在 Git 跟踪集合中，并且 merge 从 `master` 侧带回了一批旧式文档和 `.claude` 平台产物。

## 检查证据

| 检查项 | 结果 | 证据 | 说明 |
|---|---|---|---|
| process 路由健康 | PASS | `uv run --python 3.11 meta-flow workspace check --project-root .` | `process_link_health: ok`。 |
| `docs/design` | PASS | `ls -ld`; `readlink -v` | symlink 到 `../../meta-flow-artifacts/process/quant-lab/docs/design`。 |
| `docs/features` | PASS | `ls -ld`; `readlink -v` | symlink 到 `../../meta-flow-artifacts/process/quant-lab/docs/features`。 |
| merge 来源 | PASS | `git log --oneline --graph --max-count=20 --all` | HEAD 为 `234b3d3 Merge master into precheck redesign baseline`。 |
| merge 对目录的影响 | FINDING | `git diff --name-status 234b3d3^1 234b3d3`; `git diff --name-status 234b3d3^2 234b3d3` | 相对当前分支父提交，`docs/design` / `docs/features` 未变；相对 `master` 父提交，merge 结果新增这些 symlink。 |
| 被 ignore 但仍 tracked | FINDING | `git ls-files -i -c --exclude-standard` | `.claude/**`、`AGENTS.md`、`checkpoints`、`docs/design`、`docs/features`、`docs/quality`、`process` 仍被跟踪。 |
| merge 带回的旧式文档 | FINDING | `git diff --name-status 234b3d3^1 234b3d3 -- docs/qmt docs/goldminer docs/release .claude` | `docs/qmt/**`、`docs/goldminer/**`、`docs/release/{DEPLOY-CHECKLIST,FEEDBACK,MIGRATION,RELEASE-NOTES,ROLLBACK}.md`、`.claude/**` 从 master 侧进入当前树。 |

## 影响判断

| 目录 / 路径 | 当前状态 | 风险 | 建议 |
|---|---|---|---|
| `docs/design` | symlink，目标健康，但仍 tracked | 中 | 后续 CR 中决定是否 `git rm --cached docs/design`，保留本地 symlink。 |
| `docs/features` | symlink，目标健康，但仍 tracked | 中 | 同上。 |
| `docs/quality` | symlink，目标健康，但仍 tracked | 中 | 与 design/features 同批治理。 |
| `process` / `checkpoints` | symlink，路由健康，但仍 tracked | 中 | 同批治理，避免未来 merge 再把 process 路由当成源码内容。 |
| `.claude/**` / `AGENTS.md` | `.gitignore` 已声明忽略，但仍 tracked | 高 | 需要明确是否从源码仓库取消跟踪，或反向修订 `.gitignore`。 |
| `docs/qmt/**` / `docs/goldminer/**` | merge 从 master 带回普通文件 | 中 | 逐项判定公开文档 vs 内部归档；内部项迁回 artifact repo 或从 Git 取消跟踪。 |
| `docs/release/*.md` | 部分是公开 release 入口，部分历史 CR release 文档是 symlink / artifact | 中 | 保留顶层五个公开 release 入口；CR-specific release 文档继续 artifact 化。 |

## 后续整改计划建议

建议启动 `FU-CR120-001 Merge-Induced Directory Routing Regression Hardening`，按 standard CR 处理：

1. CP2：冻结范围，只做目录路由事实和目标清单确认；不执行删除、不执行 `git rm --cached`。
2. CP5：确认最小实现切片，文件 owner 包含 `.gitignore`、Git index 路由项、必要 guardrail / checker；明确是否处理 `.claude/**`、`AGENTS.md`、`docs/qmt/**`、`docs/goldminer/**`。
3. CP6：在批准后执行 index-only 清理或 guardrail hardening；不得删除本地 artifact symlink 目标内容。
4. CP7：验证 `git ls-files -i -c --exclude-standard` 不再出现已整改路径，`meta-flow workspace check` 仍 PASS，`docs/design` / `docs/features` 仍可读。
5. CP8：确认 READY，并记录保留公开文档、迁出内部文档、不授权 runtime/NAS/credentials/trading/publish。

## 不授权范围

- 不执行 `git rm --cached`、删除、移动、覆盖、checkout、reset 或清理。
- 不访问 NAS、凭据、账户、交易、provider、lake、catalog。
- 不启动 runtime。
