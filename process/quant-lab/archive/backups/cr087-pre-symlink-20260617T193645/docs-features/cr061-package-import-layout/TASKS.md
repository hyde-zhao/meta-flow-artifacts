---
status: "draft-for-batch-b-authorization"
version: "1.0"
change_id: "CR-061"
feature_id: "cr061-package-import-layout"
created_at: "2026-06-15T08:41:42+08:00"
created_by: "host-orchestrator"
implementation_allowed: false
---

# CR061 Package / Import / Layout Tasks

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-15 | host-orchestrator | 新增 Batch B 授权前任务拆解；所有实现任务 pending。 |

## Batch B 授权前任务

| task_id | 状态 | 任务 | 产物 / 验证 |
|---|---|---|---|
| CR061B-TASK-001 | DONE | 冻结 dirty worktree include/exclude 边界 | `docs/release/CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml` |
| CR061B-TASK-002 | DONE | 冻结 Batch B candidate manifest 和 source hashes | `docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml` |
| CR061B-TASK-003 | DONE | 输出 Feature design / test plan / tasks | `docs/features/cr061-package-import-layout/` |
| CR061B-TASK-004 | DONE | 输出 CR061 专属 Feature design matrix | `docs/design/FEATURE-DESIGN-MATRIX-CR061.md` |
| CR061B-TASK-005 | IN_REVIEW | 发起 CP5 Batch B 实现授权门禁 | `process/checkpoints/CP5-CR061-BATCH-B-IMPLEMENTATION-AUTHORIZATION.md` |

## Batch B 授权后候选实现任务

以下任务当前不得执行。只有用户对 CP5 Batch B checkpoint 回复 `approve` 后，才可按门禁范围继续。

| task_id | 状态 | 任务 | 限制 |
|---|---|---|---|
| CR061B-TASK-101 | DONE | 修改 `pyproject.toml` `[project].name` 为 `quant-lab` | 不改版本号、不改依赖策略。 |
| CR061B-TASK-102 | DONE | 执行 `uv lock` 再生成 `uv.lock` | 不手工编辑 lock。 |
| CR061B-TASK-103 | DONE | 新增最小 `quant_lab` namespace | 不移动 legacy roots。 |
| CR061B-TASK-104 | DONE | 视源码检查结果新增可选 alias modules | 不触发 provider/lake/runtime。 |
| CR061B-TASK-105 | DONE | 新增 import smoke test | 只做离线 import。 |
| CR061B-TASK-106 | DONE | 运行离线验证并写 CP6/CP7 evidence | CP6 PASS，CP7 PASS_WITH_RISK。 |

## 明确禁止任务

| task_id | 禁止任务 | 原因 |
|---|---|---|
| CR061B-BLOCK-001 | 移动 / 重命名 / 删除 `engine/**`、`market_data/**`、`strategies/**`、`trading/**` | 需要后续 bulk relayout CR。 |
| CR061B-BLOCK-002 | 批量 rewrite imports | 当前候选清单未授权。 |
| CR061B-BLOCK-003 | 删除 legacy roots | 破坏 compatibility-first。 |
| CR061B-BLOCK-004 | Git remote / push / tag / history rewrite | 属于 Git cutover CR。 |
| CR061B-BLOCK-005 | NAS / data lake / provider / catalog | 属于外部数据迁移 CR。 |
| CR061B-BLOCK-006 | 读取凭据或启动 QMT/MiniQMT | runtime 未授权且 CR046 暂停。 |
