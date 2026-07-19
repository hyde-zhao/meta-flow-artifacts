---
status: ready-for-human-gate
version: "0.5.0-proposed"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# CR-051 Feedback

## 1. 反馈回流入口

| Feedback ID | 类型 | 来源 | 摘要 | 分流 | follow-up 候选 | 状态 |
|---|---|---|---|---|---|---|
| FB-AW-001 | tech-debt | review | capability registry 缺失导致用户文档 claims checker 无法运行 | canonical docs owner | yes | candidate |
| FB-AW-002 | tech-debt | review | canonical module boundary manifest 缺失 | architecture owner | yes | candidate |
| FB-AW-003 | platform-gap | test | Windows native Git/durability/lock/no-follow 未 pilot | platform QA | yes | candidate |
| FB-AW-004 | reliability | test | selector writer crash 可能遗留 lock directory | design/recovery | yes | candidate |
| FB-AW-005 | security | review | 不可信并发目录迁移需要 dirfd/openat 级设计 | future migration CR | yes | candidate |
| FB-AW-006 | migration | user/project | 首个项目真实目录/软链接/worktree cutover | per-project authorized migration | yes | awaiting-project-selection |
| FB-AW-007 | operations | user | 人工 main↔integration 同步成本 | backlog/advisor | yes | observe-only |

这些条目不是已创建的 CR；只有 Host Orchestrator 在后续人工决策后写入正式 follow-up tracking 台账，才可推进。

## 2. 交付后观察计划

| Signal ID | 观察信号 | 观察方式 | 触发阈值 | 分流 |
|---|---|---|---|---|
| OBS-AW-001 | worktree switch 留下 durable intent 或驻留分支异常 | 只读 health/status + 用户报告 | 任意 1 次不可自动恢复 | defect / recovery review |
| OBS-AW-002 | 双 leg 2/2 PASS 与整体状态不一致 | aggregate record/readback | 任意 1 次 | BLOCKING defect |
| OBS-AW-003 | sibling 项目 owned path 被读取或写入 | routing/migration manifest | 任意 1 次 | security defect |
| OBS-AW-004 | selector lock 长期残留 | writer fail-closed diagnostics | 同项目连续 2 次写入被同一锁阻断 | recovery follow-up |
| OBS-AW-005 | capacity estimator 高频 fail-closed | switch precheck result | 10 次尝试中 3 次估算器失败 | estimator fixture/calibration |
| OBS-AW-006 | 文档 capability 声明漂移 | capability-claims checker | registry 恢复后任意 1 项 FAIL | documentation defect |
| OBS-AW-007 | 人工同步成为主要运维负担 | 用户记录 | 单项目连续 3 个 CR 均需额外重复同步处置 | advisor / conditional helper proposal |

## 3. 风险分流

| Risk | 类别 | 推荐 | 备选 | 重访条件 |
|---|---|---|---|---|
| 部分独立 QA | risk_acceptance | CP8 接受并如实标注 | 独立复验 ST-AW-001/002 后再终验 | 对外声明要求全量独立 QA |
| remote/branch protection 未验证 | not_authorized | 首次真实接入时单独授权 pilot | 本 CR 扩围并暂停 CP8 | 选择真实 remote |
| Windows pilots | follow_up_candidate | 首次 Windows 项目前完成 | 当前 Linux-only 继续 fail closed | 声明 Windows 支持前 |
| real migration | not_authorized | 每项目独立 CR/授权 | 不迁移，继续兼容旧布局 | 用户选择项目后 |
| TOCTOU | follow_up_candidate | 不可信并发场景前 full-LLD | 限制为可信静态输入 | 输入威胁模型变化 |

## 4. 台账边界

本文件只收集反馈和候选，不替代正式 follow-up tracking 台账，不创建新 CR，也不授权真实 Git、worktree、迁移、链接或发布动作。

## 5. CP8 R2 候选收敛（2026-07-19）

| Candidate | 状态 | 说明 |
|---|---|---|
| FU-CR051-007 | CLOSED-IN-CURRENT-CR | ST-AW-002 design delta、proof/owner/calibration/phase 已补；独立 QA 分离仍作为风险接受而非伪造关闭 |
| FU-CR051-008 | CLOSED-IN-CURRENT-CR | ST-AW-003 四个高优测试与 exact CAS cleanup 已补 |
| FU-CR051-009 | CLOSED-IN-CURRENT-CR | ST-AW-004 PARTIAL/target policy/dependency-DAG 已补 |

FU-CR051-001..006 与 FB-AW-001..007 保持原 candidate/observe-only 状态。
