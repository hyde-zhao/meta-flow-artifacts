请审查人工门禁 `CP3-CR051-ARTIFACT-WORKTREE-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR051-ARTIFACT-WORKTREE-HLD-REVIEW.md`

自动预检结论: `PASS`，17/17 项通过，0 FAIL，0 WAIVED，0 blocker。

Context Capsule: `process/context/CP3-CR051-DESIGN-CONTEXT.yaml`，`read_profile=compact`；机器上下文为 `process/context/CP3-CR051.context.json`。

审批者摘要:

- 本次确认服务的整体目标: 冻结 project-first artifact worktree、异构双 leg、聚合完成、create-only integration 和 CR 外人工 main/integration 同步架构，并使非原子 branch switch 可判定、可恢复、fail closed。
- 推荐动作: `approve`，接受下列三项推荐架构；当前设计已通过自动一致性检查且无 blocker。
- approve 后会发生什么: 进入 story-planning，由 meta-se 生成 Feature Design、正式 Story 和 Development Plan，执行 CP4，并在全量设计证据收敛后停在 CP5 人工门。
- approve 不授权什么: 不授权源码实现、真实迁移/软链接变更、真实 Git worktree/branch/ref/remote/main-sync mutation、commit/push/publish、force/history rewrite、凭据/runtime/SaaS/production write/trading 或 sibling project 修改。
- 不确认会阻塞什么: CR-051 保持在 solution-design/CP3，不进入 Feature/Story planning、CP4、LLD 或实现。

决策收集覆盖: 已扫描 STATE/prior gate、meta-se return、CP3 result、discussion checkpoint、五份设计产物和当前用户输入 6 类来源；发现 4 类架构/实现候选，其中 3 项去重后纳入人工决策，1 项作为 fail-closed 的后续 Feature Design 细节。

决策分层:

- 必须用户决策: 3
- 高风险策略确认: 2（非原子恢复、远端 ref/共享 main 边界）
- agent 默认处理: 1（checkout write-bytes 估算器，无法证明时不切换）
- 仅审计记录: 8（CP2 已批准基线、自动检查、调度、read expansion、零真实 mutation，以及 superseded CP2 result 的既有 Artifact Doctor 分类器债务等）

本轮待人工决策项: 3

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP3-CR051-DQ-01 | architecture | 非原子 worktree switch 如何安全恢复？ | worktree 外 durable intent + 完整 precheck + 一次 switch + fresh post-observation；仅满足 clean/no-Git-op/original OID 稳定/权限空间复检时条件式回 integration，否则保留现场并 `RECOVERY_REQUIRED/BLOCKED`。 | 禁用自动 switch，改为人工操作/检查；禁止退出码-only 与 reset/clean/stash/force/delete。 | 推荐保留自动化且可恢复，代价是状态机/fixture 较复杂；备选简单但用户旅程退化。 | 文件完整性、恢复幂等、磁盘/权限/中断故障；错误实现可能丢数据或产生假 PASS。 |
| CP3-CR051-DQ-02 | architecture | operation/leg/aggregate/receipt 如何单写并避免 OID 自引用？ | shared envelope + typed append-only evidence DAG；journal/leg/aggregate 各有唯一 writer，以 CR/logical attempt/required-leg set 关联，publication OID 后置绑定。 | Feature 独立 schema；或后续外部不可变账本；拒绝单一可变 lifecycle state。 | 推荐统一、可复算、无新服务；代价是 schema/owner 契约较密。备选适合独立发布或成熟外部账本。 | 证据真实性、陈旧 attempt、aggregate 假完成、并发审计。 |
| CP3-CR051-DQ-03 | architecture | integration 首建和 CR 外人工同步如何处理远端竞态？ | project lock + fresh observation + exact ordinary ref update + fresh postcheck；integration 缺失才 create-only，同步显式方向/无 active artifact CR/独立授权，冲突保留两端并阻断。 | 后续 forge API/PR/merge-queue adapter；禁止 reset/rebase/force 及单 CR 隐式同步 main。 | 推荐无平台依赖、竞态可判定，但人工同步有运维成本；forge 适合强保护分支但需新凭据/高风险 CR。 | 远端 ref 覆盖、shared main 安全、多项目并发、权限与保护策略。 |

如果你回复 approve，表示你接受以上 3 项推荐方案，不表示授权以下不授权项。

不授权项: 源码或测试实现、正式 Story 之外的提前实现、真实 artifact 文件迁移、软链接变更、真实 worktree/branch/ref/remote/main-sync、commit/push/publish、force/history rewrite、凭据、runtime、SaaS、production write、trading、quant-lab 或其他 sibling project 修改。

请只回复以下三个 exact 选项之一：

- approve
- 修改: <具体修改点>
- reject
