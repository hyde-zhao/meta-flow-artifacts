---
status: ready-for-human-gate
version: "0.5.0-proposed"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# CR-051 Deploy Checklist

## 1. 发布前输入检查

| 输入 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Release Context Capsule | PASS | `process/release/RELEASE-CONTEXT-CR051.yaml` | full profile、READY_WITH_RISK |
| CR Test Report | PASS_WITH_RISK | `process/docs/quality/CR051-TEST-REPORT.md` | 五个 Story 全部允许推进 |
| CR Review | PASS_WITH_RISK | `process/docs/quality/CR051-REVIEW.md` | 0 open BLOCKER/HIGH |
| 用户文档 | PASS_WITH_RISK | `process/returns/CR051-DOCUMENTATION.return.json` | guardrail 通过；capability registry 缺失 |
| 真实发布授权 | NOT-AUTHORIZED | NA-CR051-001..004 | CP8 仅确认 readiness |

## 2. 发布候选快照

| 检查项 | 状态 | 证据/说明 |
|---|---|---|
| 变更范围清楚 | PASS | 5 个 Story、2 份用户文档、CR 级质量/发布摘要 |
| Story 最新结果唯一 | PASS | 5/5 最新 CP7 result 均为 PASS_WITH_RISK |
| 全仓只读回归 | PASS | Host CP8 aggregate run：680 passed + 70 subtests |
| BLOCKER/HIGH 收敛 | PASS | open BLOCKER=0；open HIGH=0；closed HIGH=2 |
| 未跟踪文件与双仓库 Git 状态 | NOT-EXECUTED-RISK | 本阶段明确禁止真实 Git/repository 操作；任何 commit/push 前必须重新检查 |
| 缓存/临时文件 | PASS_WITH_WARNING | delivery guardrail 仅报告已忽略本地 `__pycache__` |
| 敏感信息/凭据 | PASS-SCOPE | 本轮未读取或使用凭据，未调用托管平台 API |

## 3. 源码仓库 leg 检查

| Check ID | 检查项 | 当前结果 | 真实执行前要求 |
|---|---|---|---|
| SRC-001 | 从最新 default branch 创建 CR | FIXTURE/CONTRACT PASS | 人工确认 default OID 与 clean state |
| SRC-002 | CR 完成目标为 default branch | CONTRACT PASS | 独立 merge/push 授权与保护规则检查 |
| SRC-003 | source leg finish evidence | PASS | 与 artifact leg 使用相同 CR ID |
| SRC-004 | commit/push/merge | NOT-AUTHORIZED | CP8 后另行授权，不能由本 checklist 自动执行 |

## 4. artifact 仓库 project-first 检查

| Check ID | 检查项 | 当前结果 | 真实执行前要求 |
|---|---|---|---|
| ART-001 | shared main 保留为共享基线 | CONTRACT PASS | 不作为项目 idle/CR owned branch |
| ART-002 | 项目长期 integration 存在且最新 | FIXTURE/CONTRACT PASS | 人工同步完成后核对 exact ref/default OID |
| ART-003 | CR 从最新项目 integration 派生 | FIXTURE PASS | clean worktree、空间、权限、journal precheck |
| ART-004 | 空闲 integration / 活动 CR 驻留 | FIXTURE PASS_WITH_RISK | 真实 worktree switch 另行授权 |
| ART-005 | CR 只回项目 integration | CONTRACT PASS | 不得直接 refresh/merge shared main |
| ART-006 | main↔integration 双向同步 | NOT-AUTHORIZED | CR 外人工操作，逐次授权与审查 |
| ART-007 | artifact commit/push/merge | NOT-AUTHORIZED | 与 source leg 分开核验、按整体交付要求配对留证 |

## 5. 双 leg 聚合检查

| Check ID | 条件 | 结果 |
|---|---|---|
| AGG-001 | source/artifact 使用相同 CR ID | PASS |
| AGG-002 | 状态固定优先级/最差状态聚合 | PASS（16/16） |
| AGG-003 | invalid/unpublished/stale handle fail closed | PASS |
| AGG-004 | 仅 2/2 PASS 可投影整体完成 | PASS |
| AGG-005 | 真实双 remote 完成聚合 | NOT-EXECUTED-RISK |

## 6. 迁移与平台矩阵

| 平台/对象 | 场景 | 当前结果 | N/A / 风险原因 |
|---|---|---|---|
| Linux temporary fixture | routing/worktree/aggregate/preflight | PASS_WITH_RISK | 不等于真实项目环境 |
| Windows native | Git/durability/lock/no-follow | NOT-EXECUTED-RISK | 当前环境非 Windows |
| 真实项目目录 | read-only migration preflight | NOT-AUTHORIZED | 需选择项目并授权读取范围 |
| 真实 artifact cutover | copy/move/link/rebranch | N/A-NOT-AUTHORIZED | 本 CR 无自动迁移，不执行写操作 |
| package install/upgrade/idempotency | installer surface | N/A | 本 CR 未改变安装器或安装路径 |

## 7. CP8 发布结论

| 项目 | 内容 |
|---|---|
| release_artifact_profile | full |
| release_decision | READY_WITH_RISK |
| 阻断项 | 0 |
| 风险接受 | 独立 QA 覆盖上限、offline fixture 边界 |
| 不授权 | 所有真实 Git/worktree/ref/remote/sync/migration/link/commit/push/publish |

## 8. 执行停止线

若任何真实操作被提出，必须停止使用本 CP8 readiness 作为授权，转由 Host Orchestrator 收集精确目标、clean-state/容量/权限检查、运行授权、回滚条件和人工确认。`approve` 不能替代这些独立授权。

## 9. CP8 R2 增量门（2026-07-19）

| 检查项 | 结果 |
|---|---|
| ST-AW-002 CP6/CP7-R2 | PASS / PASS_WITH_RISK |
| ST-AW-003 CP6/CP7-R3 | PASS / PASS_WITH_RISK |
| ST-AW-004 CP6/CP7-R2 | PASS / PASS_WITH_RISK |
| design delta | 3/3 merged |
| 全仓回归 | 697 passed、70 subtests passed |
| 新增实现 blocker/high | 0 / 0 |
| 真实外部 mutation | 0 |

R2 不改变停止线：CP8 approve 仍不是 runtime authorization。
