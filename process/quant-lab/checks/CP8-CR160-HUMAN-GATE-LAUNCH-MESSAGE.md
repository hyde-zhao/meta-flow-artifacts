请审查人工门禁 `CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-RELEASE-READINESS`。

checklist 路径: `process/checkpoints/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-RELEASE-READINESS.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-RELEASE-READINESS.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 4
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 4
blocking / high-risk 决策摘要: DQ-CP8-CR160-001, DQ-CP8-CR160-002, DQ-CP8-CR160-003, DQ-CP8-CR160-004

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR160-001 | risk_acceptance | 是否接受 CP7 `PASS_WITH_RISK` 并以 design-only `READY_WITH_RISK` 关闭 CR160？ | Approve：接受 `R-CR160-CP7-DESIGN-ONLY`、`R-CR160-FUTURE-CHECKER-DEFERRED`、`R-CR160-RUNTIME-AUTHORIZATION-LEAK` 的受控风险，关闭设计-only Stage 4 workflow。 | A. 修改后重提 CP8；B. 回退 CP3 补设计；C. 先启动 checker/schema CR 再关闭。 | 推荐方案保持最小闭环，风险显式可追踪；A/B 延迟关闭；C 扩大当前 CR 范围。 | 接受后 CR160 可 delivered with risk；拒绝会阻塞 closure。 |
| DQ-CP8-CR160-002 | runtime_authorization | 是否确认 CP8 approve 不授权任何代码/checker/runtime/data/paper/simulation/live/trading/broker/provider/NAS/catalog/Git/publish 操作？ | Approve deny-by-default non-authorization boundary。 | A. 单独启动 high-risk runtime authorization CR；B. reject CP8，要求 runtime readiness 先证明。 | 推荐方案符合 CR160 scope 和安全边界；A/B 都需要独立授权与新证据。 | 高风险；防止 `READY_WITH_RISK` 被误读为 paper/simulation/live readiness。 |
| DQ-CP8-CR160-003 | follow_up_tracking | 是否接受 6 个 follow-up candidates 作为候选，不在本轮启动？ | Approve：记录到 `process/changes/CR-160-FOLLOW-UP-TRACKING-2026-07-09.md` 和 `FEEDBACK.md`，不创建 active CR。 | A. 立即提升某个 candidate 为 active CR；B. 删除 selected candidate；C. 暂缓跟踪。 | 推荐方案保留路线且不扩大 CR160；A 需要新的 CP0/CP2；B/C 会降低后续可追踪性。 | 影响 strategy remediation、real-data validation、paper gate、runtime authorization、data-lake governance 和 checker/schema 的后续路由。 |
| DQ-CP8-CR160-004 | scope | 是否确认 CR160 关闭边界止于 Stage 4 design/gate contract，而不是 Stage 5 implementation？ | Approve：CR160 关闭 Stage 4 workflow design；Stage 5 paper/simulation gate 保持 follow-up。 | A. 回退 CP3 扩大到 Stage 5 HLD；B. 保持 CR160 active 直到 Stage 5 gate 另行设计。 | 推荐方案与 CP2/CP3 纯设计路线一致；A/B 会混合不同授权边界。 | 影响 CR160 close condition 和后续 Stage 5 baseline。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
