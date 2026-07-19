---
document: CR051 Artifact Worktree Architecture Decisions
version: 0.2.0
status: confirmed
date: 2026-07-18
cr_id: CR-051
confirmed: true
confirmed_by: "user-current-instruction"
confirmed_at: "2026-07-18T05:46:40Z"
hld_ref: process/docs/design/CR051-ARTIFACT-WORKTREE-HLD.md
discussion_ref: process/discussions/CP3-CR051-HLD-DISCUSSION-LOG.md
---

# CR-051 Artifact Worktree 架构决策草案

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1.0 | 2026-07-18 | meta-se-critical | 形成 project-first、非原子恢复、异构 leg、聚合证据、bootstrap、manual-sync 与迁移边界七项核心 ADR 草案。 |
| 0.2.0 | 2026-07-18 | meta-se-critical | 按用户当前指令批准 ADR-AW-001..007；把 capacity、durable store、manual-sync ops 固化为 O-AW-01..03 CP5/follow-up 契约。 |

## 1. 决策状态说明

- 用户当前指令已于 `2026-07-18T05:46:40Z` 批准 DQ-01..03 推荐方案和 ADR-AW-001..007；全部 ADR 状态为 `approved`。
- `ADR-AW-001/004/005/006` 分别落实 `CP3-CR051-DQ-01/02/03`；其中 005/006 共用 DQ-03 的远端竞态和同步边界决策。
- `ADR-AW-002/003/007` 承接已批准 CP2 产品边界，但仍须随整体 HLD 通过 CP3。
- CP3 批准不授权真实 Git/worktree/link/ref/remote/main-sync mutation、凭据、publish 或 production write。

## ADR-AW-001：非原子 worktree switch 采用 durable intent 与 fresh observation

- 状态：`approved`
- 决策项：`CP3-CR051-DQ-01`
- 影响面：FEAT-AW-02、文件完整性、恢复、运维、验证

### 背景

`git switch`、index/worktree 写入和进程终止不构成原子事务；命令退出码不能证明 symbolic HEAD、HEAD OID、registration 和 clean 状态已一致。

### 决策

采用 `PRECHECKED -> INTENT_DURABLE -> SWITCH_ATTEMPTED -> OBSERVATION_REQUIRED -> VERIFIED_TARGET | VERIFIED_ORIGINAL | ROLLBACK_ELIGIBLE | RECOVERY_REQUIRED`。intent 必须位于目标 worktree 外；任何 mutation、超时或错误后都重新观察。只有 clean、无 Git operation、original integration ref/OID 未漂移、权限与空间复检通过时才允许先写 rollback intent 再回退；其余情况保留现场并 `RECOVERY_REQUIRED/BLOCKED`。

禁止自动 `reset --hard`、`git clean`、stash、force、删 branch 或覆盖用户文件。

空间证明按 O-AW-01 执行：主路径输出相同观测下确定的 enumeration coverage、checkout upper bound 与 `1.5x` safety；无权限/无法枚举/误差未知均 fail closed。512MiB 只有在 CP5 bounded profile 证明 `1.5x upper_bound <= 512MiB` 且 false-safe=0、underestimate=0 后才能作为保守 floor/fallback，否则禁用 auto switch。

durable store 按 O-AW-02 执行：目标 worktree 外、store-local temp、write、file fsync、atomic replace、parent-dir fsync、checksum/readback 全部成功后才允许 Git mutation。跨设备不得依赖 rename；任一步失败 `BLOCKED`。CP5 fault fixtures 至少覆盖 ENOSPC、EACCES、fsync/replace failure、torn/corrupt record、process kill、cross-device path，并证明恢复幂等。

### 备选与切换条件

| 方案 | 优点 | 代价 / 风险 | 结论 | When to switch |
|---|---|---|---|---|
| 禁用自动 switch，仅输出人工检查步骤 | 实现和授权面最小 | 用户旅程降级，无法自动 begin/finish | 可执行治理备选 | journal 无法可靠持久化或 CP5 无法证明恢复幂等时 |
| 仅记录命令结果 | 实现简单 | 无法判定崩溃后真实状态 | 拒绝 | 不切换；不满足 CP3-DC-01 |
| 自动 reset/clean/stash | 表面恢复快 | 可能丢失用户数据 | 禁止 | 无切换条件 |

### 后果与验证

需要 project lock、空间/权限 probe、fault injection 和重复 resume fixture；收益是每个 attempt 可审计、未知现场不会被误报为 PASS。

## ADR-AW-002：使用 existing control checkout + sibling project worktree 的 project-first 拓扑

- 状态：`approved`（产品边界已由用户在 CP2 解决，CP3 当前指令确认）
- 影响面：FEAT-AW-01/02、portable routing、sibling isolation

### 决策

保留 shared artifact control checkout 作为发现/管理入口；每个 project 在 configurable sibling root 下拥有长期 worktree、独立 index/checkout/branch role。持久 metadata 只保存 anchor + relative path；设备绝对路径只是 runtime resolved value。project identity、common Git dir、branch role 和 owned-path 一致性是 hard gate；sparse checkout 仅优化可见性/容量。

### 备选与切换条件

| 方案 | 优点 | 代价 / 风险 | 结论 | When to switch |
|---|---|---|---|---|
| bare control repo + sibling worktrees | control 不占 branch | 需要真实 control 迁移，扩大当前 CR | 延后备选 | pilot 证明 existing control 持续造成不可接受的 branch 占用时另开 CR |
| 单共享 worktree + sparse 项目切换 | 目录少 | 共享 index/branch，dirty 与并发跨项目耦合 | 拒绝 | 不切换；违背 project-first 隔离 |

### 后果与验证

多解、identity/layout 冲突一律 `BLOCKED`；sibling dirty 既不读取也不阻断。验证覆盖跨设备相对路径、sibling negative path 和 owned-path 外零写入。

## ADR-AW-003：source/artifact 使用异构 completion target，并显式覆盖 CR-050 paired-default 假设

- 状态：`approved`
- 影响面：FEAT-AW-03/04、CR-050 compatibility、CR completion

### 决策

source leg 从/回 fresh source default；shared-artifact leg 从/回 `projects/<project>/integration`。artifact CR branch 为 `projects/<project>/cr/<cr-id>-<slug>`；单 CR 不 refresh、merge 或更新 shared main。mode=`shared-artifact-project-first` 时，策略层必须拒绝 artifact default/main merge plan。

继续复用 CR-050 的 argv-only、ref validation、exact OID、fresh observation、typed authorization、append-only attempt、dry-run 和非破坏性 cleanup contract；只覆盖 paired-default target/projection 假设。

### 备选与切换条件

| 方案 | 优点 | 代价 / 风险 | 结论 | When to switch |
|---|---|---|---|---|
| artifact CR 仍回 shared main | 与旧 paired-default 统一 | 跨项目耦合，扩大单 CR 权限并恢复 R2 语义 | 拒绝 | 不切换；除非产品基线通过新 CR 重开 |
| source/artifact 都只完成到各自 CR branch | 最保守 | 无稳定 integration/default 完成语义，后续人工负担高 | 治理备选 | 远端策略暂时禁止 completion target update 时 |

### 后果与验证

需要 mode applicability contract test；artifact main mutation count 必须为 0，任一旧 paired-default plan 在 shared-artifact mode 下必须 fail closed。

## ADR-AW-004：证据使用共享 envelope + typed append-only DAG，aggregate 单写

- 状态：`approved`
- 决策项：`CP3-CR051-DQ-02`
- 影响面：FEAT-AW-02/03/04、审计、并发、兼容

### 决策

Worktree journal 由 lifecycle writer 单写；source/artifact leg result 分别由对应 executor 追加；aggregate result 仅由 aggregate coordinator 写。对象通过 CR ID、logical attempt、required-leg set 和稳定 evidence ref 组成单向 DAG：`intent -> observation/leg result -> aggregate -> optional publication binding`。

内容 digest 排除自身 digest 字段；receipt 不包含承载自身的未来 artifact commit OID，该 OID 只能由更晚的 binding event 关联。aggregate 是纯验证/投影边界，按 `BLOCKED > FAIL > IN_PROGRESS > PASS`，且仅全 required legs PASS 才 PASS；`PARTIAL` 只属于 progress/effect。

### 备选与切换条件

| 方案 | 优点 | 代价 / 风险 | 结论 | When to switch |
|---|---|---|---|---|
| 每 Feature 独立 schema | 局部简单 | correlation 漂移，聚合难验证 | 条件备选 | Feature 形成真正独立产品/发布生命周期并拆 HLD 时 |
| 单一可变 lifecycle state | 查询简单 | 覆盖历史、多写竞争、不可证明时序 | 拒绝 | 不切换 |
| 外部不可变账本 | 强审计能力 | 新依赖、迁移和运行授权 | 延后备选 | 外部账本能提供同等单写、稳定 ID 与离线 fixture 时 |

### 后果与验证

CP5 前必须证明 writer ownership、schema correlation、无环引用、可复算 aggregate 与 stale/missing result 拒绝路径；否则回退 CP3。

## ADR-AW-005：integration bootstrap 仅执行 exact-OID create-only

- 状态：`approved`
- 决策项：`CP3-CR051-DQ-03`
- 影响面：FEAT-AW-02、远端竞态、初始化幂等

### 决策

在 project lock 下 fresh observe remote integration。若存在，只验证 identity/role，禁止 recreate/reset/orphan；若缺失，从 fresh `origin/main` exact OID 发起一次 ordinary create-only update，并 fresh postcheck。并发拒绝后若实际 integration 等于 seed OID，返回 `NO_CHANGE`；若不同或无法观察，返回 `BLOCKED`。

### 备选与切换条件

| 方案 | 优点 | 代价 / 风险 | 结论 | When to switch |
|---|---|---|---|---|
| forge API / merge queue 创建 | 可服从远端保护策略 | 需凭据、平台契约和独立授权 | 条件备选 | 远端禁止 ordinary exact create 时另开高风险 adapter CR |
| reset/rebase/force 对齐 | 快 | 改写现有 integration 事实 | 禁止 | 无切换条件 |

### 后果与验证

需要并发同值/异值 fixture、remote query failure 和重复 bootstrap 测试；任何不确定状态不得重试 force 或报告 PASS。

## ADR-AW-006：shared main ↔ integration 同步是 CR 外显式人工 operation

- 状态：`approved`
- 决策项：`CP3-CR051-DQ-03`
- 影响面：CR-external sync coordinator、权限、运维、共享 main

### 决策

同步必须显式选择 `MAIN_TO_INTEGRATION` 或 `INTEGRATION_TO_MAIN`，默认要求项目无 active artifact CR、取得 project lock、fresh observe 两端 exact OID、验证 fast-forward eligibility，并使用与 target/expected OID 绑定的独立 typed authorization。执行一次 ordinary exact update 后 fresh postcheck；冲突或不确定时保留两端并 `BLOCKED`，不自动反向重写。

### 备选与切换条件

| 方案 | 优点 | 代价 / 风险 | 结论 | When to switch |
|---|---|---|---|---|
| forge PR/merge queue | 审批可见、适合保护分支 | 凭据与平台依赖，超出本 CR | 条件备选 | 团队要求所有 main update 必须走远端审批时 |
| 单 CR finish 隐式同步 main | 自动化程度高 | 与 active CR 竞态，扩大授权和回滚面 | 拒绝 | 不切换；需新 CR 重开产品边界 |
| force/rebase 自动解冲突 | 快 | 改写历史、损失事实 | 禁止 | 无切换条件 |

### 后果与验证

CP3/CP8 approval 都不授予该 operation 的 runtime authority。验证仅使用 local bare remote fixture；真实 shared remote pilot 后置。

O-AW-03 要求每次 manual-sync 记录频率、人工耗时、terminal 与阻塞原因。单项目每周 `>=3` 次且连续 `4` 周，或人工同步中位耗时 `>10` 分钟，或可避免的调度阻塞率 `>5%`，任一触发后只创建独立“条件式同步助手”CR candidate；本 ADR 不允许 CR-051 自动同步。

## ADR-AW-007：migration 只交付 read-only manifest 与人工 handoff

- 状态：`approved`
- 影响面：FEAT-AW-05、迁移、回滚、交付授权

### 决策

本 CR 仅生成 legacy→project-first mapping、hash/count、link plan、readiness、验证与回滚 checklist；真实 file/link/worktree/ref mutation、commit 和 publication 次数为 0。manifest PASS 只表示具备后续项目级人工迁移输入，不代表已迁移或获授权。

### 备选与切换条件

| 方案 | 优点 | 代价 / 风险 | 结论 | When to switch |
|---|---|---|---|---|
| 本 CR 自动迁移真实 artifact | 一次交付 | 扩大 destructive scope、回滚和 sibling 风险 | 拒绝 | 不切换；必须独立 migration CR 与授权 |
| 不交付 manifest | 范围更小 | 用户无法安全接续逐项目迁移 | 不推荐 | 仅目标项目明确永久保持 legacy layout 时 |

### 后果与验证

manifest 必须可重复生成且无副作用；路径/hash/count 不一致时 readiness=`BLOCKED`，不得静默搬移或创建 link。

## 2. ADR 一致性与落地映射

| ADR | HLD / Blueprint 映射 | 主要验证 | CP3 状态 |
|---|---|---|---|
| ADR-AW-001 | HLD §5.1；FLOW-AW-03；CP3-DC-01 | capacity/durable-store fault matrix + recovery idempotency | approved / DQ-01 |
| ADR-AW-002 | HLD §4；FEAT-AW-01/02 | portable route、sibling/owned-path negative | approved |
| ADR-AW-003 | HLD §5.3；FLOW-AW-04 | mode target、artifact-main-zero-write | approved |
| ADR-AW-004 | HLD §5.3/§6；Evidence DAG | schema/correlation/single-writer/aggregate table | approved / DQ-02 |
| ADR-AW-005 | HLD §5.2；IntegrationBootstrapAttempt | create race same/different OID | approved / DQ-03 |
| ADR-AW-006 | HLD §5.4；ManualSyncAttempt | active-CR/authz/ff-only + ops thresholds | approved / DQ-03 |
| ADR-AW-007 | HLD §4/§9.1；FEAT-AW-05 | dry-run/no-mutation/hash-count | approved |

## 2.1 CP5 / Follow-up 开放契约

| Open ID | 状态 | 必须证明 | 失败 / 触发路由 |
|---|---|---|---|
| O-AW-01 | `non-blocking-open` | deterministic capacity upper bound、1.5x safety、bounded 512MiB fallback、0 false-safe/underestimate；含常见、误差、无权限/无法枚举 fixture | 禁用 auto switch |
| O-AW-02 | `non-blocking-open` | store-local temp + fsync/replace/dir-fsync/checksum/readback；ENOSPC/EACCES/corruption/kill/cross-device 下 0 提前 mutation、恢复幂等 | Git mutation 前 `BLOCKED` |
| O-AW-03 | `non-blocking-open` | manual-sync frequency/duration/blocker metrics | 达任一固定阈值只创建条件式同步助手 CR candidate |

## 3. 平台适配与 Feature Design 触发

- 当前推荐只依赖本地 native Git/文件系统和既有 typed policy/evidence adapter；不假设特定 forge。
- 远端强制 PR/merge queue 时，ADR-AW-005/006 切换为后续 forge adapter CR；本 HLD 不自行使用凭据或 SaaS。
- ADR-AW-001、003、004、005、006 涉及状态、跨模块 contract、权限、并发或外部 ref，必须进入 CP3 后 Feature DESIGN/TEST-PLAN/TASKS；ADR-AW-002/007 的 schema 与 deny-mutation contract 也需 Feature Design。

## 4. Gotchas

1. `approved` 只确认架构；仍不得据此执行 mutation，且 O-AW-01/02 的 CP5 失败会禁用 auto switch。
2. project lock 只解决本地单写，不替代远端 exact OID 与 fresh postcheck。
3. journal、result、receipt 是证据，不是读取即执行的恢复命令。
4. shared-artifact mode 与 CR-050 paired-default artifact target 不能同时生效。
5. rollback 条件不成立时，保留现场并阻断是设计成功，不是“恢复失败后继续”。
6. manual-sync 和 migration 都需要独立授权；它们不能从 CP3 `approve` 继承 runtime authority。

## 5. CP3 建议

ADR-AW-001..007 与 DQ-01..03 推荐方案已获用户当前指令批准。Host 可据此回填 CP3 approval 并进入下一阶段规划；该批准不授权任何真实运行或外部写入。O-AW-01/02 路由 CP5，O-AW-03 路由运维观察/follow-up candidate。
