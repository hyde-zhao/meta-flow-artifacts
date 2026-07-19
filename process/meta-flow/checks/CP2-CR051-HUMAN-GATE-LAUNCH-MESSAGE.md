# CR-051 CP2 人工门发起消息

请审查：`process/checkpoints/CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE.md`

自动预检结论：`PASS`；CP1/CP2 blockers=0、waivers=0。

Context Capsule：`process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml`（ready，read_profile=compact）；机器 context pack `process/context/CP2-CR051.context.json` 已通过校验。

## 审批者摘要

- 本次确认服务的整体目标：在共享 artifact Git 仓库中建立 project-first 命名空间、每项目独立 worktree 和项目作用域 Git 生命周期。
- 推荐动作：`approve`，接受下方三项推荐方案并进入 CP3 详细设计。
- approve 后会发生什么：调度 meta-se 生成 Blueprint/HLD/ADR 和 CP3 人工门；不会直接修改源码。
- approve 不授权什么：不授权跳过 CP3/CP5，不授权源码实现、真实 artifact 迁移、软链接变更、真实 worktree/branch/ref/remote mutation、force、凭据或 runtime/production 操作。
- 不确认会阻塞什么：CR-051 停在 requirement-clarification，不启动正式架构设计、Story、LLD 或实现。

决策收集覆盖：已扫描 current state、CR/route plan、meta-pm return、CP0/CP1/CP2 result、discussion、八份产品基线、用户显式选择与源码事实共 8 类来源；4 个 Scenario Gray Areas 中 1 个已由用户解决、3 个纳入待决策；缺失阻断来源 0。

本轮待人工决策项：3

## 决策分层

- 必须用户决策：3
- 高风险策略确认：1（共享 main refresh 的 branch 历史策略；本门不授权真实 Git 写）
- agent 默认处理：4（字段名、错误码措辞、fixture 名称、局部模块命名）
- 仅审计记录：6（已确认方向、真实调度、零真实 mutation 等）

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP2-DQ-01 | architecture | worktree 长期常驻、每 CR 临时还是 hybrid？ | 长期常驻 per-project worktree；active 使用 project-namespaced branch；idle detached 细节由 CP3 冻结。 | A：每 CR 临时；B：persistent 加高风险临时的 hybrid。 | 推荐方案保证软链接和恢复入口稳定；代价是 stale/磁盘/安全 remove 治理。 | 影响 CLI、metadata、branch 占用、迁移手册和测试。 |
| CP2-DQ-02 | architecture | 共享 main 前进后如何刷新当前项目 branch？ | 显式 merge fresh origin/main 到当前 project CR branch；冲突 fail closed；不 rebase/force/自动解冲突。 | A：rebase 加 force-with-lease；B：fresh main 重建后受控 cherry-pick。 | 推荐方案不改写历史且无需 force；代价是可能产生 merge commit。 | 影响 OID 证据、publish/merge/finish、冲突恢复和历史可读性。 |
| CP2-DQ-03 | architecture | control repo 与 project worktree 默认拓扑？ | 保留现有 control checkout；worktree 放在可配置 sibling root；使用 project namespace/sparse policy 和相对 metadata。 | A：bare control；B：sibling full worktrees 加 owned-path gate。 | 推荐方案兼容现有 clone 和逐项目迁移；代价是需防误用 control 并验证 sparse。 | 影响 link/check、discovery、portable metadata、回滚和迁移。 |

完整待决策表、备选优劣和回退条件见上述 checkpoint。

如果你回复 approve，表示接受 CP2-DQ-01..03 的全部推荐方案；不表示授权任何不授权项。

不授权项：

- 源码实现或跳过 CP3/CP5；
- 真实 artifact 文件迁移或软链接变更；
- 真实 worktree、branch、commit、tag、stash 或 ref 创建/删除；
- remote fetch/push/default update/delete、force、forge API 或凭据读取；
- runtime、SaaS、production write、publish 或 trading。

请直接回复以下任一整行：

approve

修改: <具体修改点>

reject
