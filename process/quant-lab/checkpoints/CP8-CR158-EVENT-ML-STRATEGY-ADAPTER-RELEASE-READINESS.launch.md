请审查：`process/checkpoints/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.md`

自动预检结论：PASS。CP8 release-readiness result 已生成，manual_gate_status=pending；CP7 结论为 `PASS_WITH_RISK`，blocker_count=0，剩余风险 1 项。

审批者摘要：本次确认服务的整体目标：确认 CR158 Event + ML Strategy Adapter 合并 CR 是否可以按当前 local/static/fixture 交付范围进入 CP8 release readiness 收口。

推荐动作：批准 `READY_WITH_RISK`。

上下文胶囊：`process/context/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-CONTEXT.yaml`
Context Capsule Summary：capsule=`process/context/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-CONTEXT.yaml`；read_profile=compact；默认读取策略为 release context、CP8 result、CP8 checkpoint；全文档读取仅在审计冲突、字段缺失或用户要求时触发。

决策收集覆盖：已扫描 CP7 result/evidence/return、verification/test/review/fixes、release context/docs、CP5 accepted decisions、development plan、story status；候选问题 6 个，纳入待决策 3 个；真实发布执行、真实 event feed、真实 ML training、真实 runtime、registry/publish/trading 授权为 N/A，因为本 CP8 不请求这些权限。

Decision Collection Coverage：scanned_sources=11；candidate_questions=6；included_pending_decisions=3；n/a_or_missing_reason=真实发布执行、真实 feed/training/runtime/registry/publish/trading authorization not requested in this CP8 gate.

决策分层：
| 层级 | 数量 | 处理 |
|---|---:|---|
| 必须用户决策 | 1 | DQ-CP8-CR158-001 |
| 高风险策略确认 | 1 | DQ-CP8-CR158-002 |
| agent 默认处理 | 1 | DQ-CP8-CR158-003 |
| 仅审计记录 | 1 | install / migration N/A |

本轮待人工决策项：3

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR158-001 | risk_acceptance | 是否接受 `R-CR158-CP7-STATIC-FIXTURE-ONLY` 并批准 `READY_WITH_RISK` | approve `READY_WITH_RISK` | reject and reopen CP7 / CP6 | 推荐方案优点是与 CP7 证据一致且不扩大权限；缺点是保留 static/fixture 风险。备选更保守但会阻塞 closure。 | approve 可关闭本地 adapter slice；reject 会阻塞 closure | 若后续需要真实数据/runtime，另开 CR 和授权门 |
| DQ-CP8-CR158-002 | runtime_authorization | 是否继续禁止真实 feed/training/provider/lake/NAS/credential/runtime/trading/registry/publish/Git remote 操作 | approve deny-by-default | grant separate explicit runtime gate | 推荐方案优点是边界清晰且风险低；缺点是不能证明真实运行。备选可推进真实验证但需要新授权。 | 推荐方案避免把 fixture PASS 误解为生产准备 | 只有用户另行明确授权才切换 |
| DQ-CP8-CR158-003 | follow_up_tracking | 是否把真实 event feed / ML training / registry / runtime 验证留作后续独立事项，不在 CR158 自动启动 | approve deferred follow-up only | promote to new CR now | 推荐方案优点是当前 CR 可收口；缺点是真实验证仍未开始。备选可提前讨论真实验证但会扩大当前门禁范围。 | 推荐方案避免扩大本轮 CR 范围和权限 | 当用户明确要求真实验证时创建新 CR 或 runtime gate |

如果你回复 approve：表示接受以上 3 项推荐决策，CR158 可关闭为 `READY_WITH_RISK` 当前交付。

approve 后会发生什么：我会回填 CP8 人工审查结果、更新 checkpoint / gate / CR ledger、关闭 CR158 当前交付。

approve 不授权什么：不表示授权真实发布、Git remote write、数据湖 / NAS / provider 访问、凭据读取、QMT/gateway runtime、simulation/paper/live/trading、broker 操作、catalog/store/registry/model/feature/label/prediction 写入、external framework 运行或生产可用性声明。

不确认会阻塞什么：不确认会阻塞 CR158 closure。

不授权项：真实发布；Git remote write；真实 event feed/listener；真实 ML training / model registry；数据湖 / NAS / provider / credential；QMT/gateway runtime；simulation/paper/live/trading/broker；catalog/store/registry/model/feature/label/prediction write；external framework run；生产可用性声明。

请回复以下三种之一：
approve
修改: <具体修改点>
reject
