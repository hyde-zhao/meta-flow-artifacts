---
discussion_id: "CP3-CR051-HLD-DISCUSSION"
change_id: "CR-051"
phase: "solution-design"
status: "ready-for-cp3-decision"
owner: "meta-se"
created_at: "2026-07-18T04:10:00Z"
context_ref: "process/context/CP3-CR051-DESIGN-INPUT.context.json"
checkpoint_ref: "process/checks/CP3-CR051-DISCUSSION-CHECKPOINT.json"
---

# CR-051 CP3 Architecture Gray Areas Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se | 基于已批准 CP2 R3、CP3-DC-01、27 项 REQ-AW 与 15 项 TC-AW，形成 CP3 前置架构灰区和 Decision Brief 输入。 |

## 讨论边界

- 本日志是 HLD 方案形成输入，不替代 HLD、ADR 或 CP3 人工 checkpoint。
- 用户已要求直接推进到 CP3 人工门；因此能形成安全候选的真实取舍统一转为 `decision-item`，不在 CP3 前重复追问。
- CP2 已冻结 project-first、existing control checkout + configurable sibling worktree、长期 project integration、短期 project CR branch、异构双 leg、单一 aggregate gate、create-only bootstrap 与 CR 外人工同步；CP3 不重新打开这些产品边界。
- 当前阶段只设计。真实 Git/worktree/link/remote/main-sync mutation、源码实现、Story/LLD 和 sibling project 读取均不授权。

## AGA-CR051-01：非原子 worktree switch 的恢复协议

**为什么影响 HLD**：`git switch`、索引/工作树写入和进程终止不构成原子事务。若只按退出码推进，会把中间态误报为 `active-cr` 或 `idle-integration`，直接违反 CP3-DC-01。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. worktree 外 durable intent + fresh observation + conditional rollback（推荐） | 状态可判定、可重入；崩溃后不依赖进程内存；允许在安全条件成立时自动回 integration | 增加 journal phase、互斥和恢复测试 | 状态 / 文件系统 / Git / 可靠性 / 验证 | 推荐；形成 `CP3-CR051-DQ-01` | 受控状态目录和 fsync/atomic-replace 可用；若持久化失败则 switch 前 fail closed |
| B. 只记录命令结果，失败后人工检查 | 实现最少 | 命令成功不等于事务成功；崩溃后无法区分实际分支；不满足 CP3-DC-01 | 安全 / 恢复 / 审计 | 不推荐 | 仅可作为缩小范围的治理备选：本轮禁用自动 switch |
| C. 每次失败都自动 reset/clean/stash 后回退 | 表面恢复快 | 会覆盖用户文件、扩大 destructive authority，且仍无法证明现场 | 安全 / 数据完整性 | 禁止 | 无切换条件；与 CP2/CP3-DC-01 冲突 |

**推荐协议**：`precheck -> INTENT_DURABLE -> switch attempt -> OBSERVATION_REQUIRED -> VERIFIED_TARGET`；异常时先 fresh observe。只有 worktree clean、无进行中的 Git operation、original integration ref/OID 未漂移、权限和空间复检通过，才写 rollback intent 并尝试回 integration；否则进入 `RECOVERY_REQUIRED/BLOCKED`。禁止自动 `reset --hard`、`clean`、stash、force、删 branch 或覆盖文件。

**分类**：`decision-item`；CP3 DQ=`CP3-CR051-DQ-01`。

## AGA-CR051-02：operation / leg / aggregate / receipt 的证据模型和单写边界

**为什么影响 HLD**：两个异构 leg 可以独立结束，但只有一个协调者能判定逻辑 CR；若 result、aggregate 和 receipt 相互回写或相互包含自身 commit OID，会产生循环引用、覆盖历史或多写竞争。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 共享 envelope + typed payload + append-only attempt DAG（推荐） | correlation 一致；schema 可独立演进；aggregate 可从 leg refs 复算；避免第二套 ledger | 需要明确 owner、digest 排除字段和校验顺序 | 数据 / 审计 / 恢复 / 可测试性 | 推荐；形成 `CP3-CR051-DQ-02` | 既有 result/ledger adapter 可复用；若无法保证单写则 CP5 前阻断 |
| B. 每个 feature 自定义完整 schema | 局部自治 | 重复 CR/attempt/OID 语义，长期漂移；aggregate 难以验证 | 维护 / 兼容 | 不推荐 | 只有不同 feature 具有独立产品/发布生命周期时另立 HLD |
| C. 单一可变 branch-lifecycle state 文件 | 查询简单 | 覆盖 attempt 历史，多 agent 写竞争，恢复无法证明先后 | 审计 / 并发 | 不推荐 | 不作为当前备选 |

**推荐边界**：Worktree Operation Journal 仅由 project worktree lifecycle writer 单写；每个 leg result 由对应 leg executor 追加；aggregate result 仅由 aggregate coordinator 写；receipt 只记录已观察事实。内容摘要计算时排除自身 digest 字段。承载 receipt 的后续 Git commit OID 不写回 receipt，而由更晚的 binding/ledger event 关联，形成单向 DAG：`intent -> leg results -> aggregate -> optional publication binding`。

**分类**：`decision-item`；CP3 DQ=`CP3-CR051-DQ-02`。

## AGA-CR051-03：integration create-only CAS 与 CR 外人工 main/integration 同步

**为什么影响 HLD**：integration 首次创建和人工同步都写远端 ref，存在远端竞态；如果复用单个 artifact CR 的权限或自动 reset/rebase，会覆盖其他项目/CR 事实。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. project-scoped lease + exact OID ordinary ref update + fresh postcheck（推荐） | 不需要 forge API；竞态可判定；远端拒绝原样保留；无 force | 无法提供远端原子多 ref 事务；人工同步仍有运维成本 | Git / 权限 / 并发 / 恢复 | 推荐；形成 `CP3-CR051-DQ-03` | 远端接受 ordinary exact update；不接受时 BLOCKED，不绕过策略 |
| B. 依赖 forge API/merge queue 做同步 | 可利用保护策略和 receipt | 需要凭据、平台契约和独立授权；超出本 CR | 安全 / 平台 / 交付 | 条件备选 | 远端强制 PR/queue 且用户另开高风险 CR 时切换 |
| C. 自动 reset/rebase/force-with-lease 对齐 | 操作快 | 改写历史、扩大权限、与产品边界冲突 | 数据完整性 / 安全 | 禁止 | 无切换条件 |

**推荐协议**：bootstrap 先 fresh `ls-remote`。integration 缺失时，以 fresh `origin/main` exact OID 做一次非 force create-only update；被并发拒绝后重新观察，若新 ref 等于 seed OID 则 `NO_CHANGE`，若不同则 `BLOCKED`。人工同步是独立 operation，显式方向 `MAIN_TO_INTEGRATION | INTEGRATION_TO_MAIN`，默认要求项目无 active artifact CR、获取 project-scoped lock、绑定 before/expected OID 与独立授权；冲突时保留两端并进入人工恢复。

**分类**：`decision-item`；CP3 DQ=`CP3-CR051-DQ-03`。

## AGA-CR051-04：control/project worktree、portable metadata 与 sparse/owned-path 隔离

**为什么影响 HLD**：共享 object database 不等于共享写入面。若 control checkout、project worktree、namespace 和 ownership 的身份无法一致校验，当前项目命令可能污染 sibling project。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. existing control + sibling project worktree + portable registry + owned-path hard gate（推荐） | 延续 CP2 已批拓扑；跨设备可恢复；每项目独立 checkout/index/branch | metadata 和 health 校验较多；sparse 不能当安全边界 | 路由 / 文件 / Git / 兼容 / 维护 | 推荐；按已批准边界落实 | project identity、common git dir、anchor-relative path、branch role 和 owned paths 全部一致 |
| B. bare control repo + sibling worktrees | control 不占 branch，拓扑更纯 | 需要真实 control 转换和迁移，扩大本 CR 操作面 | 运维 / 迁移 | 延后备选 | 真实 pilot 证明 existing control 持续造成 branch 占用时另开 CR |
| C. 单共享 worktree + sparse 切换项目 | 少目录 | 共享 index/branch，切换和 dirty 状态跨项目耦合 | 并发 / 安全 | 不推荐 | 不适用于当前产品基线 |

**推荐边界**：canonical metadata 只保存 anchor + relative path；设备绝对路径只能是运行时解析值。owned-path 是硬门，sparse checkout 只用于可见性/性能，不提供安全保证。任何 identity/path/layout/branch-role 多解均 `BLOCKED`；sibling dirty 不读取也不阻断。

**分类**：`resolved-by-user`（CP2-DQ-03、SGQ-AW-001/003）；实现字段细节为 agent 默认处理，CP5 可复核。

## 方案形成结论

| 输入来源 | 结论 | 影响的正式产物 |
|---|---|---|
| 用户批准 CP2 R3 | 冻结 project-first、异构双 leg、全 PASS 聚合、create-only bootstrap、CR 外人工同步 | Blueprint、Domain Map、HLD、ADR 全部章节 |
| 用户追加 CP3-DC-01 | switch 不得被称为原子；恢复必须 durable、fresh-observation、fail-closed | Domain state machine、HLD §关键流程/失败路径、ADR-AW-001 |
| lane-product（meta-se 综合） | 不重新打开迁移、main refresh 或真实 pilot 范围 | HLD 非目标、Release slices |
| lane-architecture（meta-se critical） | 采用 Option A：在既有 typed lifecycle 上增加 project context、recoverable worktree protocol 与 aggregate gate | 候选方案、模块、Dependency Map |
| lane-quality（meta-se 综合） | 两项目临时 bare remote + crash/fault injection + negative command spy 是 CP7 最低验证面 | HLD NFR、验证矩阵、Feature triggers |
| lane-docs-check | 手工恢复必须基于实际观测，禁止给出破坏性“快捷修复” | HLD 运维/恢复、ADR Gotchas |

## DQ 分类汇总

| 灰区 | 分类 | DQ / 证据 | 阻断 CP3 自动预检 |
|---|---|---|---|
| AGA-CR051-01 | decision-item | CP3-CR051-DQ-01 | 否；存在安全推荐与可执行治理备选 |
| AGA-CR051-02 | decision-item | CP3-CR051-DQ-02 | 否；单写与无自引用方案可验证 |
| AGA-CR051-03 | decision-item | CP3-CR051-DQ-03 | 否；远端拒绝可 fail closed |
| AGA-CR051-04 | resolved-by-user | CP2-DQ-03、SGQ-AW-001/003 | 否 |

未发现 `non-blocking-open`、`converted-to-spike`、`n/a-with-reason` 或无法形成安全方案的 blocker。
