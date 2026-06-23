---
status: "draft-for-batch-b-authorization"
version: "1.0"
change_id: "CR-061"
created_at: "2026-06-15T08:41:42+08:00"
created_by: "host-orchestrator"
implementation_allowed: false
---

# FEATURE-DESIGN-MATRIX-CR061

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-15 | host-orchestrator | 新增 CR061 专属 Feature 设计矩阵，用于 Batch B 实现授权门禁。 |

## 1. Feature 判定矩阵

| feature_id | Feature | status | design_required | design_ref | test_plan_ref | tasks_ref | reason |
|---|---|---|---|---|---|---|---|
| cr061-package-import-layout | Package / Import / Layout Convergence | required | yes | `docs/features/cr061-package-import-layout/DESIGN.md` | `docs/features/cr061-package-import-layout/TEST-PLAN.md` | `docs/features/cr061-package-import-layout/TASKS.md` | 触及 package metadata、lockfile、import namespace 和 regression boundary，必须有 Feature design。 |

## 2. Story / LLD 策略

| story_id | Story | feature_design_refs | lld_policy.required_level | 当前状态 | 说明 |
|---|---|---|---|---|---|
| CR061B-S01 | Dirty worktree include/exclude freeze | `cr061-package-import-layout` | technical-note | done-for-authorization | 以 YAML manifest 固化，不改源码。 |
| CR061B-S02 | Batch B candidate manifest freeze | `cr061-package-import-layout` | technical-note | done-for-authorization | 固化 action、hash、blocked action。 |
| CR061B-S03 | Metadata and lock convergence | `cr061-package-import-layout` | full-lld | pending-authorization | 后续若获批才改 `pyproject.toml` / `uv.lock`。 |
| CR061B-S04 | Compatibility namespace | `cr061-package-import-layout` | full-lld | pending-authorization | 后续若获批才新增 `quant_lab/**`。 |
| CR061B-S05 | Offline import smoke validation | `cr061-package-import-layout` | technical-note | pending-authorization | 后续若获批才新增 / 执行 smoke test。 |

## 3. 设计证据覆盖

| 覆盖项 | 证据 |
|---|---|
| 架构方案 | `docs/release/REPO-LOCAL-PACKAGE-LAYOUT-PLAN-CR061.md` |
| 候选对象 | `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md` |
| Batch B action manifest | `docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml` |
| Dirty worktree 边界 | `docs/release/CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml` |
| Preserve-audit | `docs/release/PRESERVE-AUDIT-DECISIONS-CR061.md` |
| Rollback reference | `docs/release/ROLLBACK-REF-CR061.md` |
| CP5 Batch B 门禁 | `process/checkpoints/CP5-CR061-BATCH-B-IMPLEMENTATION-AUTHORIZATION.md` |

## 4. 不授权声明

本矩阵不授权真实实现。Batch B 真实实现必须等待 `process/checkpoints/CP5-CR061-BATCH-B-IMPLEMENTATION-AUTHORIZATION.md` 人工审查结果为 approved。
