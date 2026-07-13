请审查人工门禁 `CP8-CR166-DELIVERY-READINESS`。

checklist 路径: `process/checkpoints/CP8-CR166-DELIVERY-READINESS.md`
自动预检结论: PASS；5/5 Story 已验证，仓库全量 1986/1986 通过，open findings=0，waivers=0。

审批者摘要:
- 本次确认服务的整体目标: 验收 fixture/static-only Walk-forward/OOS typed C2 producer foundation，并在不启动 Stage 3、不连接真实数据或 runtime 的前提下关闭 CR-166。
- 推荐动作: approve，并以 READY_WITH_RISK 关闭 CR-166。
- approve 后会发生什么: CR-166 标记 closed/cp8_closed；Stage 2 保持 complete；CR-166 记录为 Stage 2→Stage 3 桥接增强；当前工作流进入 delivered。
- approve 不授权什么: 不授权 commit/push/tag/publish/deploy、真实 fold/OOS、真实数据、lake/NAS/provider、credentials、external framework、runtime、broker、simulation、paper/live/trading、catalog/store/registry write 或 Stage 3 启动。
- 不确认会阻塞什么: CR-166 保持 active/cp8_pending；已实现源码、测试和证据保留。

Context Capsule: `process/context/CP8-CR166-DELIVERY-CONTEXT.yaml`，read_profile=minimal。
决策收集覆盖: CP6/CP7、4 份质量报告、Release Context、Stage/authorization claim 和既有 follow-up 均已扫描。
决策分层:
- 必须用户决策: 1
- 高风险策略确认: 0
- agent 默认处理: 0
- 仅审计记录: 3
本轮待人工决策项: 1
blocking / high-risk 决策摘要: DQ-CP8-CR166-001

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR166-001 | risk_acceptance | 是否接受 same-host inline fallback 不具备独立 agent/model 隔离，并关闭 CR-166？ | 接受并以 READY_WITH_RISK 关闭 | 保持 active，未来用独立 reviewer 复核 CP7 | 推荐可立即收敛且证据完整；备选隔离更强但推迟关闭 | 只影响验证组织独立性声明，不改变 1986/1986、0 failed、0 open finding 的事实 |

如果你回复 approve，表示接受以上 1 项推荐方案，不表示授权任何真实数据、runtime、Stage 3 或外部/远端动作。
不授权项: commit/push/tag/publish/deploy、真实 fold/OOS、真实数据、lake/NAS/provider、credentials、external framework、runtime、broker、simulation、paper/live/trading、catalog/store/registry write、Stage 3 启动。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
