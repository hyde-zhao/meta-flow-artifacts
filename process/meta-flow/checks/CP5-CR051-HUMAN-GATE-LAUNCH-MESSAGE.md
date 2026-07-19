请审查人工门禁 `CP5-CR051-ALL-STORIES-LLD-BATCH`。

checklist 路径: `process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md`

自动预检结论: `PASS`。5/5 设计证据、5/5 Story CP5 result、CP4 DAG、安全与审计检查均通过；R1 F01/F02/F03 全部关闭，R2 独立复核新增 finding=0，blocker=0，waiver=0。

Context Capsule: `process/context/CP5-CR051-LLD-CONTEXT.yaml`，`read_profile=minimal`；全文档读取以 `deep_review` 入账，当前 read-log 校验 OK。

审批者摘要:

- 本次确认服务的整体目标: 批准 project-first artifact worktree 的完整 5 Story 可实现设计，让异构 source/artifact 双 leg 在后续 CP6/CP7 中实现和验证。
- 推荐动作: `approve`，接受当前 4 full LLD + 1 technical-note、DAG/Wave/file-owner、R2 closure 和不可豁免 proof obligations。
- approve 后会发生什么: 标记全量设计证据 confirmed，进入 story-execution，先按 W1 调度 ST-AW-001，再依次推进 W2/W3/W4；每个 Story 仍需 CP6/CP7。
- approve 不授权什么: 不授权真实 Git/worktree/ref/remote/link/migration/main-sync、commit/push/publish、凭据/SaaS/生产写入/交易或 sibling project 修改。
- 不确认会阻塞什么: 5 个 Story 的 Dev Gate 保持关闭，CR-051 不进入源码实现。

决策收集覆盖: 已扫描 prior gates、CP4 result、5 个 Story result、R1/R2 independent review、O-AW 跟踪项、全量设计批次与权限边界；候选问题 11 个，去重后仅 1 个需要本门人工决定，其余为已批准、已修复、不可豁免验证义务、agent fail-closed 默认或当前不授权项。

决策分层:

- 必须用户决策: 1
- 高风险策略确认: 1（O-AW-01/02、public port、evidence DAG、2/2 aggregate hard gates）
- agent 默认处理: 3（CAP/DUR 失败 manual-only；aggregate 非 2/2 不投影；O-AW-03 candidate-only）
- 仅审计记录: 9

本轮待人工决策项: 1

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP5-CR051-DQ-01 | implementation | 是否批准完整设计批次并进入实现？ | approve；按 W1→W2→W3→W4，先 ST-AW-001，每个 Story 仍过 CP6/CP7，硬门不得 waiver。 | 点名 Story/契约修改；公共契约变化回 CP4/CP3；或 reject。 | 推荐方案已通过机器与独立复核；点名修改控制返工但可能重规划；reject 停止交付。 | false-safe、durability、port 漂移、证据自引用与假完成；以 initial-disabled、fail-closed、2/2 matching PASS、artifact-main mutation=0 控制。 |

完整待决策表、回退条件和 CP6/CP7 proof obligations 见 checkpoint 文件。

如果你回复 approve，表示接受本轮 1 项推荐方案，并授权进入 Story 实现调度；不表示授权任何真实仓库操作。

不授权项: 真实 Git/worktree/ref/remote/link/migration/main-sync、commit、push、publish、凭据、SaaS、生产写入、交易和 sibling project 修改。

可回复：`approve`、`修改: <具体修改点>` 或 `reject`。
