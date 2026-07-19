---
status: "ready-for-cp4"
version: "1.0"
active_change_ref: "CR-051"
source_blueprint: "process/docs/design/CR051-ARTIFACT-WORKTREE-BLUEPRINT.md"
source_hld: "process/docs/design/CR051-ARTIFACT-WORKTREE-HLD.md"
source_adr: "process/docs/design/CR051-ARTIFACT-WORKTREE-ARCHITECTURE-DECISION.md"
context_ref: "process/context/CP4-CR051-STORY-PLANNING-CONTEXT.yaml"
confirmed_by: ""
confirmed_at: ""
---

# CR-051 Feature Design Matrix

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | CP3 批准后建立 5 个 required Feature、5 个稳定 Story 的实现设计矩阵；固定 ST-AW-001..004 full-lld、纯只读 ST-AW-005 technical-note，并将 O-AW-01/02/03 分别路由到 ST-AW-002/005。 |

## 适用性判定规则

| 维度 | required 触发条件 | waived / n/a 条件 |
|---|---|---|
| 数据与状态 | 新增 portable route、worktree journal、leg/aggregate result、migration manifest 或状态机 | 纯展示且无 schema、状态或兼容语义 |
| 接口与依赖 | ProjectContext、WorktreeHealth、LegResult、AggregateResult 等跨模块契约需冻结 | 单文件局部改动且无相邻调用方 |
| 权限与安全 | Git/ref/worktree、typed authorization、owned-path、deny-mutation 或外部同步边界 | 不改变权限、写入面或安全门 |
| 运行与可靠性 | 并发锁、非原子 switch、durable intent、expected OID、幂等、恢复与聚合 | 无运行时风险、重试或失败恢复 |
| 多 Story 复用 | route/context/evidence 契约被 2 个以上 Story 消费 | 单 Story 且可由正式 technical-note 完整覆盖 |

## Feature 设计矩阵

| Feature ID | Feature / Epic | 来源 | 适用性 | 判定理由 | 需要产物 | 关联 Story | 建议 lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|---|
| FEAT-AW-01 | Project Context & Portable Routing | CAP-AW-01；ADR-AW-002；REQ-AW-001..003 | required | portable schema、legacy/new 唯一 write target、project identity 与跨模块 RouteDecision 必须先冻结 | `process/docs/features/cr051-routing/{DESIGN,TEST-PLAN,TASKS}.md` | ST-AW-001 | full-lld | 新 resolver 不能在不破坏旧 process route 的前提下提供单向兼容时回 CP3/设计澄清 |
| FEAT-AW-02 | Recoverable Project Worktree Lifecycle | CAP-AW-02；ADR-AW-001/002/005；O-AW-01/02 | required | 非原子 switch、project lock、integration create-only、capacity proof、durable intent 与故障恢复均为高风险状态/并发契约 | `process/docs/features/cr051-worktree/{DESIGN,TEST-PLAN,TASKS}.md` | ST-AW-002 | full-lld | O-AW-01 或 O-AW-02 在 CP5 无法证明时禁用 auto switch；不得降级为破坏性恢复 |
| FEAT-AW-03 | Heterogeneous Leg Lifecycle | CAP-AW-03；ADR-AW-003/004；REQ-AW-008..011/016 | required | source-default / artifact-integration target override、typed authz、expected OID、独立 LegResult 与 CR-050 适用性需冻结 | `process/docs/features/cr051-legs/{DESIGN,TEST-PLAN,TASKS}.md` | ST-AW-003 | full-lld | 若实现必须触碰 artifact main、继承 paired-default target 或跨 leg 自动回滚，返回设计澄清 |
| FEAT-AW-04 | Aggregate Gate & Evidence | CAP-AW-04；ADR-AW-004；REQ-AW-012/016/017 | required | overall 单写、result correlation、无环 evidence、固定优先级及 CR/state/current projection hard gate 属跨 Story 核心契约 | `process/docs/features/cr051-aggregate/{DESIGN,TEST-PLAN,TASKS}.md` | ST-AW-004 | full-lld | 无法证明 matching required legs 2/2 PASS 才投影或出现多写/自引用时回 CP3 |
| FEAT-AW-05 | Migration Preflight & Handoff | CAP-AW-05；ADR-AW-006/007；REQ-AW-014..015；O-AW-03 | required | manifest schema、readiness、deny-mutation、人工回滚交接和 manual-sync 指标阈值需统一设计 | `process/docs/features/cr051-migration/{DESIGN,TEST-PLAN,TASKS}.md` | ST-AW-005 | technical-note | 一旦增加 file/link/worktree/ref/remote mutation、自动同步或跨项目读取，立即升级 full-lld 并重开范围/授权 |

## Story 下游消费表

| Story ID | feature_design_refs | lld_policy.required_level | trigger_reasons | CP5 设计证据路径 | CP5 审查方式 |
|---|---|---|---|---|---|
| ST-AW-001 | `process/docs/features/cr051-routing/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | portable-schema, legacy-write-conflict, cross-module-contract | `process/stories/STORY-ST-AW-001-project-first-routing-LLD.md` | Story 自动可实现性检查 + 全量批次人工确认 |
| ST-AW-002 | `process/docs/features/cr051-worktree/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | non-atomic-state-machine, concurrency, filesystem, remote-ref, recovery | `process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md` | 同上；必须包含 O-AW-01/02 fixture 证明 |
| ST-AW-003 | `process/docs/features/cr051-legs/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | cross-repo, authorization, expected-oid, shared-contract, recovery | `process/stories/STORY-ST-AW-003-heterogeneous-git-legs-LLD.md` | 同上；与 ST-AW-004 冻结 LegResult contract |
| ST-AW-004 | `process/docs/features/cr051-aggregate/{DESIGN,TEST-PLAN,TASKS}.md` | full-lld | single-writer, correlation, evidence-dag, workflow-projection | `process/stories/STORY-ST-AW-004-aggregate-evidence-gate-LLD.md` | 同上；验证 16 组合和 non-PASS projection=0 |
| ST-AW-005 | `process/docs/features/cr051-migration/{DESIGN,TEST-PLAN,TASKS}.md` | technical-note | read-only-migration, manifest-schema, deny-mutation, follow-up-tracking | `process/stories/STORY-ST-AW-005-read-only-migration-handoff.md#技术说明` | Story technical-note 自动检查 + 全量批次人工确认 |

## CP5 / Follow-up 开放契约

| Open ID | 归属 Story / Feature | CP5 必须证明 | 失败 / 触发路由 | 状态 |
|---|---|---|---|---|
| O-AW-01 | ST-AW-002 / FEAT-AW-02 | deterministic checkout upper bound、`1.5x` safety、bounded 512MiB 仅在 `1.5x upper_bound <= 512MiB` 且 false-safe=0、underestimate=0 时启用；覆盖常见/误差/无权限/无法枚举 fixture | 禁用 auto switch；保持只读/人工路径，不新增风险接受 | non-blocking-open / CP5 evidence obligation |
| O-AW-02 | ST-AW-002 / FEAT-AW-02 | store-local temp、write/file-fsync/atomic-replace/parent-dir-fsync/checksum/readback；ENOSPC/EACCES/fsync/replace/corruption/kill/cross-device 下提前 Git mutation=0、恢复幂等 | 任一步失败在 Git mutation 前 BLOCKED；不得跨设备 rename 或 copy+delete 冒充原子替换 | non-blocking-open / CP5 evidence obligation |
| O-AW-03 | ST-AW-005 / FEAT-AW-05 | manual-sync attempt 记录频率、人工耗时、terminal/阻塞原因和可避免调度阻塞；三个固定阈值可计算 | 单项目每周 >=3 次且连续 4 周，或中位耗时 >10 分钟，或可避免阻塞率 >5%，任一命中只创建条件式同步助手 follow-up CR candidate | non-blocking-open / follow-up-tracking |

## 提前确认的关键决策

CP3-CR051-DQ-01..03 与 ADR-AW-001..007 已批准；本矩阵未发现新的 architecture、security、runtime_authorization 或 risk_acceptance 决策。O-AW-01/02 是 CP5 可验证义务，O-AW-03 是量化 follow-up 路由，均不改变当前不授权边界，也不需要新的人工作业决策。

## 豁免与 N/A

Feature required=5、waived=0、n/a=0。Story full-lld=4、technical-note=1、waived=0。ST-AW-005 使用 technical-note 的必要条件是其实现和验证始终保持纯只读：真实文件迁移、软链接变更、worktree/ref/remote mutation、commit/push/publish 与自动 main↔integration 同步均为 0；任一条件变化即升级 full-lld 并重新评估范围和运行授权。

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 所有 Feature / Epic 均已判定 | PASS | FEAT-AW-01..05 共 5/5 required |
| required Feature 产物计划完整 | PASS | 5 组 DESIGN/TEST-PLAN/TASKS 固定路径 |
| 每个 Story 均有 refs 与 lld_policy | PASS | ST-AW-001..005 共 5/5；4 full-lld + 1 technical-note |
| O-AW-01/02/03 已路由 | PASS | O-AW-01/02→ST-AW-002；O-AW-03→ST-AW-005 |
| 新增人工决策已收集 | PASS | 0；沿用已批准 CP3 决策且不扩大权限 |

## Gotchas

- Feature `required` 不等于对应 Story 必须 full-lld；ST-AW-005 的 Feature pack 必不可少，但纯只读 Story 可用正式 technical-note 作为 CP5 证据。
- 512MiB 不是未知 repo 的默认放行值；未满足 bounded profile 和 0 false-safe/underestimate 证明时必须禁用 auto switch。
- `PARTIAL` 只能表达 progress/effect；不得加入 aggregate overall 枚举或被投影为逻辑 CR 完成。
- Feature Design、CP4 或 CP5 的批准都不授权真实 Git/worktree/ref/remote/link/migration/main-sync mutation。
