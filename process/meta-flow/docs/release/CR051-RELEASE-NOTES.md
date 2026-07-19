---
status: ready-for-human-gate
version: "0.5.0-proposed"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# CR-051 Release Notes

## 1. 摘要

| 项目 | 内容 |
|---|---|
| 当前版本 | 0.4.0 |
| 建议目标版本 | 0.5.0（仅 CP8 建议，未写入版本、未发布） |
| 发布结论 | READY_WITH_RISK |
| 发布范围 | 多项目共享 artifact 仓库的 project-first Git/worktree 生命周期、双 leg 聚合和只读迁移 preflight |
| 主要风险 | 独立 QA 覆盖上限、capability registry 缺失、真实环境/Windows/迁移未验证 |

`READY_WITH_RISK` 表示可进入人工交付终验，不表示已经发布，也不授权任何真实仓库或迁移操作。

## 2. 版本号决策

| 项目 | 内容 |
|---|---|
| 当前版本 | 0.4.0 |
| 建议目标版本 | 0.5.0 |
| 变更类型 | MINOR |
| 兼容性 | 源码仓库生命周期兼容；artifact 切换需未来按项目迁移 |
| 推荐原因 | 新增长期 project integration、短期 CR、可恢复 worktree 与双 leg 聚合等成组能力，现有 public workspace CLI 未被宣称替换 |

## 3. 新增能力与用户可见变化

| Change ID | 内容 | 用户影响 | 来源 |
|---|---|---|---|
| REL-AW-001 | artifact 仓库采用 shared main + per-project integration + project-namespaced CR branch | 每个项目拥有稳定 worktree 与独立 CR 合流边界 | ST-AW-001/002 |
| REL-AW-002 | 源码 leg 与 artifact leg 使用不同完成目标 | 源码回 default branch；artifact 回项目 integration | ST-AW-003 |
| REL-AW-003 | 双 leg 结果按最差状态聚合，2/2 PASS 才完成 | 任一侧失败、阻断或过期都不会误报整体完成 | ST-AW-004 |
| REL-AW-004 | 提供只读 migration preflight manifest 与 handoff | 可在未来迁移前先检查映射、deny、symlink 与 readiness | ST-AW-005 |

## 4. 两类仓库的生命周期

### 源码 / 项目交付仓库

CR 从最新 default branch（`main` 或 `master`）创建，完成后目标仍是 default branch。该流程不因为共享 artifact 仓库而改变。

### artifact / 过程产物仓库

保留 shared `main` 作为共享基线。每个项目维护长期 `projects/<project>/integration`，worktree 空闲时驻留该分支；每个 CR 从最新项目 integration 创建项目命名空间短期分支，活动时 worktree 切到该 CR 分支，完成后只合回项目 integration。shared main 与项目 integration 的两个方向同步都属于 CR 外人工操作。

本轮没有执行上述任何真实分支、worktree、合并或同步动作。

## 5. 行为修复

| Finding | 修复 | 结果 |
|---|---|---|
| REV-AW003-001 | 统一 default-clock freshness 基准 | CLOSED-R2 |
| REV-AW005-001 | 每对象 deny guard，禁止 denied descendant probe/mapping/leak | CLOSED-R2 |

## 6. 破坏性变更

当前代码交付没有自动迁移、强制切换或删除 legacy layout，因此不声明已发生破坏性运行时变更。未来把已有项目从共享 main/旧目录切换到 project integration 和项目优先目录时，是需要单独授权、preflight 和回滚方案的迁移事件。

## 7. 安装、升级与迁移

- 本 CR 不改变 Meta Flow 安装器目标，也没有执行安装/升级。
- ST-AW-005 只提供 Python library/API 级只读 preflight；没有 migration CLI。
- 当前执行的迁移结论为 N/A。未来迁移必须逐项目开展，详见 `process/docs/release/CR051-MIGRATION.md`。

## 8. 已知问题与风险

| Risk ID | 严重度 | 状态 | 处理 |
|---|---|---|---|
| CR051-RISK-PARTIAL-INDEPENDENT-QA | MEDIUM | risk-acceptance | ST-AW-001/002 为 bounded Host fallback，不声称全量独立 QA |
| DOC-CR051-CAPABILITY-REGISTRY-MISSING | MEDIUM | follow-up | 恢复 registry 后重跑 claims checker |
| CR051-RISK-REMOTE-UNVERIFIED | MEDIUM | not-authorized | 真实 remote/凭据/branch protection/publication 另行授权 |
| CR051-RISK-WINDOWS-NATIVE-PILOTS | LOW | follow-up | 首次 Windows 支持前 pilot |
| R-AW004-LOCK-CRASH | LOW | fail-closed | 残留锁需诊断或受控恢复 |
| R-AW005-DIR-ENUM-TOCTOU | MEDIUM | follow-up | 不可信并发迁移需 full-LLD |
| CR051-RISK-REAL-MIGRATION-UNVERIFIED | MEDIUM | not-authorized | 真实迁移按项目另行授权 |
| CR051-RISK-MODULE-BOUNDARIES-MANIFEST | MEDIUM | follow-up | 补 canonical manifest |

## 9. 回滚方式

本轮没有真实部署或迁移，所以当前无需执行回滚。未来授权切换的停止、恢复与验证步骤见 `process/docs/release/CR051-ROLLBACK.md`；该文档是程序性计划，不是执行授权。

## 10. 参考

| 类型 | 路径 |
|---|---|
| Release Context | `process/release/RELEASE-CONTEXT-CR051.yaml` |
| Verification | `process/docs/quality/CR051-VERIFICATION-REPORT.md` |
| Test Report | `process/docs/quality/CR051-TEST-REPORT.md` |
| Review | `process/docs/quality/CR051-REVIEW.md` |

## 11. CP8 R2 终验回修说明（2026-07-19）

- ST-AW-002/003/004 的终验实现、测试与设计证据缺口已在当前 CR 内关闭。
- 最新全仓回归为 697 tests + 70 subtests 全部通过。
- 新增回修验证由 Host inline-fallback 完成；用户明确禁止子 Agent，因此 release decision 继续为 `READY_WITH_RISK`。
- `FU-CR051-007..009` 已在当前 CR 关闭；`FU-CR051-001..006` 保持 candidate-only。
- 真实 remote/worktree/ref/sync/migration/link/commit/push/publish 仍未授权、未执行。
