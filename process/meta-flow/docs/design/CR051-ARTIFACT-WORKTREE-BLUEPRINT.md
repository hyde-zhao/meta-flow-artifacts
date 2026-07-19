---
status: confirmed
version: "1.1"
created_at: "2026-07-18"
owner: "meta-se"
cr_ref: "CR-051"
source_story_map: "process/docs/product/STORY-MAP.md"
source_mvp_scope: "process/docs/product/MVP-SCOPE.md"
review_gate: "CP3"
confirmed: true
confirmed_by: "user-current-instruction"
confirmed_at: "2026-07-18T05:46:40Z"
---

# CR-051 Project-first Artifact Worktree Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se | 在 CP2 R3 边界内定义 project-first 路由、每项目 worktree、异构双 leg、aggregate gate、迁移交接和非原子 switch 恢复能力。 |
| 1.1 | 2026-07-18 | meta-se-critical | 按用户当前指令确认 DQ-01..03 推荐架构；补充空间估算、durable store 与 manual-sync 运维成本三项 CP5/follow-up 契约，不授予运行权限。 |

## 1. 蓝图适用性

判定：`required`。CR-051 同时包含五个跨 Feature 能力、多个领域对象、共享 Git object database、每项目独立写入面、异构 leg、聚合状态和 CR 外同步；若直接压入 HLD，会让数据 owner、禁止依赖和 CR-050 override 失真。

本 Blueprint 只定义能力/数据/依赖边界，不冻结 Python 类、函数、CLI 参数或 Story 文件。

## 2. 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖候选 Story | Owner Feature |
|---|---|---|---|---|
| CAP-AW-01 | Project-first Routing | 当前项目唯一解析自己的 docs/process，legacy 共存时不写错 | ST-AW-001 | FEAT-AW-01 |
| CAP-AW-02 | Per-project Worktree Lifecycle | 每项目拥有稳定 checkout/index/branch，并可安全恢复非原子切换 | ST-AW-002 | FEAT-AW-02 |
| CAP-AW-03 | Heterogeneous Git Legs | source 回 source default；artifact 只回 project integration | ST-AW-003 | FEAT-AW-03 |
| CAP-AW-04 | Aggregate Completion Gate | 仅全部必需 leg PASS 完成，部分成功保留真实事实和恢复入口 | ST-AW-004 | FEAT-AW-04 |
| CAP-AW-05 | Migration Handoff | 用户获得逐项目迁移清单，而本 CR 真实迁移为 0 | ST-AW-005 | FEAT-AW-05 |

## 3. Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-AW-01 | Project Context & Portable Routing | project identity、layout version、anchor-relative route、legacy dual-read、唯一 write target、runtime resolved path | worktree/ref mutation；根据路径存在性猜 write target；迁移文件/link | `ProjectArtifactConfig`、route decision | route metadata、control/worktree registry、legacy paths | 不依赖 lifecycle executor、aggregate result、sibling project 内容 |
| FEAT-AW-02 | Recoverable Project Worktree Lifecycle | create/check/list/safe-remove、integration create-only bootstrap、branch-role health、project lock、CP3-DC-01 switch/recover | source leg 完成；aggregate 判定；自动 reset/clean/stash/force；shared-main 同步 | `ProjectWorktreeRegistration`、`WorktreeOperationJournal`、worktree health | FEAT-AW-01 context、fresh Git observations、filesystem probes | 不写 CR/state/current；不读取 sibling owned paths；不依赖 aggregate 反向驱动 |
| FEAT-AW-03 | Heterogeneous Leg Lifecycle | 按 mode 为 source/artifact 解析不同 base/target，复用 native Git safety contract，输出独立 leg result | 逻辑 CR overall；artifact main refresh；跨 leg rollback；manual sync | `LegAttempt`、`LegResult`（按 repo role 单写） | project context、worktree health、Git refs、typed authorization | artifact leg 禁止依赖 shared main 作为 target；不得调用 FEAT-AW-05 mutation |
| FEAT-AW-04 | Aggregate Gate & Evidence | 验证 CR/attempt correlation，按 `BLOCKED > FAIL > IN_PROGRESS > PASS` 聚合，仅全 PASS 放行 | 执行 Git、修复 leg、自动关闭失败 CR、把 PARTIAL 当 terminal success | `AggregateResult`、progress/effect projection | validated leg result refs、policy | 不依赖 Git adapter；不得由 artifact 单 leg 推断 overall PASS；不得触发 manual sync |
| FEAT-AW-05 | Migration Preflight & Handoff | legacy→project-first mapping、hash/count、link plan、readiness、验证/回滚 checklist | 文件移动、link/worktree/ref mutation、commit/publish | `MigrationManifest` | FEAT-AW-01 route decision、FEAT-AW-02 read-only health | 不依赖 lifecycle mutation API；不得读取 quant-lab/sibling 内容超出显式目标 |

## 4. 共享能力与唯一 owner

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-AW-01 | Native Git Probe/Executor | FEAT-AW-02/03、CR 外 sync coordinator | `meta_flow.workspace.git_sync` 边界 | 上层 typed policy → argv-only adapter | timeout/nonzero 后 fresh observe；不推断成功 |
| SH-AW-02 | Project Context Resolver | FEAT-AW-02/03/05 | FEAT-AW-01 | 上层 → resolver（read-only） | 多解/冲突 `BLOCKED` |
| SH-AW-03 | Project-scoped Operation Lock | FEAT-AW-02/03、CR 外 sync | FEAT-AW-02 lifecycle boundary | mutation coordinator → lock | 无法取得锁 `BLOCKED`；不偷锁/不清理未知锁 |
| SH-AW-04 | Append-only Evidence Adapter | FEAT-AW-02/03/04 | existing result/ledger adapter | executor/coordinator → writer | 写入失败不得推进 terminal PASS |
| SH-AW-05 | Typed Authorization Validator | leg finish、bootstrap remote create、manual sync、remove | existing authorization boundary | operation → validator | 缺失/失配/过期 `BLOCKED` |

共享能力不是新平台层：它们扩展既有 routing、native Git、result/ledger 和 authorization 边界，不新增第二套状态/ledger。

## 5. 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-AW-01 | resolve/check project | FEAT-AW-01 → FEAT-AW-02 | FEAT-AW-01 写 route decision；FEAT-AW-02 写 health | identity/layout/path 多解→BLOCKED、0 mutation | TC-AW-001..003,010 |
| FLOW-AW-02 | prepare/attach worktree | FEAT-AW-01 → FEAT-AW-02 | FEAT-AW-02 | bootstrap race、branch occupancy、nested/unknown dir→fresh observe 后 NO_CHANGE/BLOCKED | TC-AW-004,007,011,015 |
| FLOW-AW-03 | integration↔CR switch | FEAT-AW-02 | FEAT-AW-02 journal 单写 | post-observation 不确定→RECOVERY_REQUIRED/BLOCKED；只在安全条件下 rollback | CP3-DC-01；TC-AW-004,009,015 |
| FLOW-AW-04 | logical CR leg lifecycle | FEAT-AW-01/02 → FEAT-AW-03 → FEAT-AW-04 | leg executor 分别写 leg result；aggregate coordinator 单写 overall | 任一 leg 非 PASS→overall 非 PASS；不自动跨 leg rollback | TC-AW-005..010,012,014 |
| FLOW-AW-05 | migration handoff | FEAT-AW-01/02 → FEAT-AW-05 | FEAT-AW-05 | 未授权时只给 dry-run/manifest；mutation=0 | TC-AW-012,013,015 |

## 6. 分支与作用域角色

| Repo role / mode | Base | Active branch | Completion target | Shared main relation |
|---|---|---|---|---|
| source leg | fresh source remote default exact OID | source CR branch（沿用 CR-050 canonical contract） | same source default，经显式授权与证明 | source default 是本 leg target |
| shared-artifact leg | fresh `projects/<project>/integration` exact OID | `projects/<project>/cr/<cr-id>-<slug>` | same project integration | shared `main` 不是 per-CR target；divergence 本身不阻断该 CR |
| artifact idle worktree | project integration | N/A | project integration | 不 checkout shared main |
| CR-external manual sync | fresh main + integration refs | 独立 sync operation，不复用 CR branch | 显式方向的 exact target | 默认无 active artifact CR；独立授权 |

## 7. CR-050 适用性覆盖

| CR-050 contract | CR-051 shared-artifact 模式 |
|---|---|
| argv-only native Git、输入/ref 校验、exact OID、fresh observation、typed authz、append-only attempt、dry-run、no force/reset/rebase/stash | `APPLIES`，作为所有 Git 操作的共同安全底座 |
| source leg 从 fresh source default open/publish/merge/finish | `APPLIES` |
| artifact repo 与 source repo 都以各自 remote default 为 base/target | `OVERRIDDEN`：artifact base/target 改为 project integration |
| paired default merge 的 artifact→project destructive order | `NOT_APPLICABLE` 于 shared-artifact CR；artifact main 不更新。overall 仍通过异构 leg aggregate gate判定 |
| finish 依赖 2/2 paired-default merge projection | `OVERRIDDEN`：每个 leg 各自满足 completion proof；aggregate 只消费两个独立 terminal result |
| cleanup fresh reproof、recovery ref、delete 独立授权 | `APPLIES_WITH_TARGET_OVERRIDE`：artifact proof 针对 integration/CR branch，不针对 main |

两套当前语义不得同时生效：一旦 route mode=`shared-artifact-project-first`，任何 artifact default/main merge plan 都必须被策略层拒绝。

## 8. 权限与安全边界

- CP3 批准只允许进入 Story planning，不授权任何真实 Git/worktree/link/remote mutation。
- 本 CR 验证只允许临时 repo/local bare remote fixture；真实 shared artifact remote pilot 后置。
- sparse checkout 不是安全边界；owned-path、project identity、common git dir、worktree registration 和 branch role 才是 hard gate。
- result/journal/receipt 均不得触发“读取即恢复”；恢复只能由显式 operation 消费 fresh observation。
- 所有真实 mutation authority 按 operation、project、repo role、ref、expected OID 绑定，且不得从 CP approval 继承。

## 9. Feature 级实现设计触发条件

| Feature | 判定 | 触发原因 | CP3 后目标产物 |
|---|---|---|---|
| FEAT-AW-01 | required | portable schema、legacy/new write-target conflict、跨模块 route contract | `docs/features/feat-aw-01-routing/{DESIGN,TEST-PLAN,TASKS}.md` |
| FEAT-AW-02 | required | 非原子状态机、并发锁、filesystem/permission/space、rollback/recovery | `docs/features/feat-aw-02-worktree/{DESIGN,TEST-PLAN,TASKS}.md` |
| FEAT-AW-03 | required | CR-050 override、跨 repo contract、typed authz、expected OID | `docs/features/feat-aw-03-legs/{DESIGN,TEST-PLAN,TASKS}.md` |
| FEAT-AW-04 | required | aggregate schema、single writer、PARTIAL 语义、correlation | `docs/features/feat-aw-04-aggregate/{DESIGN,TEST-PLAN,TASKS}.md` |
| FEAT-AW-05 | required | migration manifest schema、deny-mutation、回滚交接 | `docs/features/feat-aw-05-migration/{DESIGN,TEST-PLAN,TASKS}.md` |

上述文件只在 CP3 approved 后由 `implementation-design` 生成；本阶段不创建。

## 10. 待确认边界

| Decision ID | 状态 | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|---|
| CP3-CR051-DQ-01 | `approved` | architecture | 非原子 switch 的 durable intent、post-observation 与 rollback 边界 | worktree 外 journal + fresh observation + conditional rollback + recovery-required | 禁用自动 switch，只提供人工操作/检查 | 推荐可恢复可审计但状态机较复杂；备选简单但用户旅程降级 | 文件安全、可靠性、维护和验证 | CP5 无法证明 durable store 或恢复幂等时禁用 auto switch |
| CP3-CR051-DQ-02 | `approved` | architecture | operation/leg/aggregate/receipt schema 与单写 | shared envelope + typed payload + append-only DAG + later publication binding | 分 feature schema | 推荐避免漂移/自引用；备选局部简单但聚合难验证 | 审计、并发、兼容 | CP5 无法证明单写或无环 refs 时回 CP3 |
| CP3-CR051-DQ-03 | `approved` | architecture | bootstrap CAS 与 CR 外人工 sync 的远端竞态/互斥 | project lock + exact ordinary ref update + fresh postcheck + explicit direction | future forge adapter | 推荐无新依赖且 fail closed；备选适合强保护远端但需凭据/新 CR | Git 并发、权限、运维 | 远端强制 PR/queue 时阻断 native path 并切后续 adapter CR |

批准来源：用户当前指令（`2026-07-18T05:46:40Z`）。该批准确认架构，不授权任何 Git/worktree/link/ref/remote/main-sync mutation。

## 10.1 CP5 / Follow-up 开放契约

| Open ID | 分类 | 契约 | 路由 / 关闭条件 |
|---|---|---|---|
| O-AW-01 | `non-blocking-open` | 空间估算须对相同观测确定、故障 fail closed；512MiB 仅在 bounded profile 证明 `1.5x upper_bound <= 512MiB` 且 0 false-safe/underestimate 后可作 floor/fallback | FEAT-AW-02 CP5 fixture；否则禁用 auto switch |
| O-AW-02 | `non-blocking-open` | intent store 使用 store-local temp、write+fsync+atomic replace+parent-dir fsync+checksum/readback；任一步失败在 Git mutation 前 BLOCKED | FEAT-AW-02 CP5 fault fixture；跨设备不得依赖 rename |
| O-AW-03 | `non-blocking-open` | manual-sync 记录频率、耗时、阻塞原因；任一阈值触发只建立条件式同步助手 CR 候选 | 单项目每周≥3 次且连续4周，或中位耗时>10分钟，或可避免调度阻塞率>5% |

## 11. Gotchas

1. 蓝图中的 project integration 是该项目 artifact lifecycle 的长期 base/target，不是 shared main 的别名。
2. `git switch` 的退出码既不是 success proof，也不是 current branch truth；必须重新观察。
3. sparse checkout 只能减少可见路径，不能替代 owned-path 和 changed/staged/committed path 检查。
4. aggregate `PARTIAL` 只能表达进度/影响；overall terminal 仍使用 `BLOCKED|FAIL|IN_PROGRESS|PASS`。
5. receipt 不能包含“承载本 receipt 的未来 commit OID”；该 OID 只能由后续 binding event 关联。
6. migration preflight 的 PASS 不是迁移、link、worktree 或 publication 授权。
