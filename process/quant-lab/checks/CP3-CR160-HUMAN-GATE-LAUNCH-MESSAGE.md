请审查人工门禁 `CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-HLD-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-HLD-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 5
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 5
blocking / high-risk 决策摘要: CP3-DQ-CR160-001, CP3-DQ-CR160-002, CP3-DQ-CR160-003, CP3-DQ-CR160-004, CP3-DQ-CR160-005

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP3-DQ-CR160-001 | architecture | 是否接受 manual layered review checklist + fail-closed post-review decision table 作为 CR160 Stage 4 推荐架构？ | Approve 方案 A：人工分层 checklist 审查 Stage 1/2/3/横切证据，再用 fail-closed decision table 输出分类和 escalation route。 | A. 现在实现 automatic checker；B. 合并 Stage 4/5 gate；C. 完全排除 contract-only artifact。 | 推荐方案最小、符合 design-only、可 CP7 验证；checker 需要代码/Story/LLD；Stage4/5 合并会扩大授权风险；排除 contract-only 会降低 remediation route 价值。 | 影响 Stage 4 workflow 主体、CP7 验证范围和后续 checker follow-up 的基线。 |
| CP3-DQ-CR160-002 | architecture | 是否确认 `observation_plan_template` 与 future Stage 3 `observation_plan_instance` 的边界？ | Approve template/instance split：CR160 定义 template；future Stage 3 package 必须产出 instance；Stage 4 审查 instance 对 template 的合规性。 | A. `observation_plan_ref` 直接指向 CR160 template；B. 暂不定义 plan；C. 当前 CR 同时定义 future instance schema。 | 推荐方案避免模板被误判为具体计划；A 会让 template-only ref 误通过；B 保持 CR157 缺口；C 会扩大到 schema/implementation 范围。 | 影响 Stage 3 handoff 合同、Stage 4 checklist、missing plan fail-closed 行为。 |
| CP3-DQ-CR160-003 | security | 是否确认 contract-only lane 永远不能输出 `paper_candidate=true`、`simulation_ready=true` 或 `runtime_authorized=true`？ | Approve fail-closed ceiling：`contract_only` 只能输出低等级 review conclusions，如 `not_reviewable`、`needs_remediation`、`needs_real_data_validation` 或 `authorization_blocked`。 | A. contract-only 完全不可进入 review；B. contract-only 可在人工风险接受后成为 observation candidate；C. 暂不声明 ceiling。 | 推荐方案允许早期 artifact 被审查和路由，但不允许 readiness overclaim；A 更保守但价值低；B/C 会放大 runtime/paper/simulation 误授权风险。 | 高风险 no-overclaim 决策；影响 Stage 5 view、paper/simulation 误读和后续自动 checker。 |
| CP3-DQ-CR160-004 | scope | 是否确认 CR155 只能作为 `blocked_admission_failed` seed sample，而不是 observation candidate？ | Approve CR155 classification：既有 readonly real-data evidence + rerun consistency 只证明确定性和 fail-closed，`BLOCKED/FAIL/paper_candidate=false` 必须分类为 `blocked_admission_failed`。 | A. 暂不使用 CR155 seed；B. 将 CR155 作为 observation candidate with risk；C. 只在 HLD 文字说明，不写 seed classification。 | 推荐方案可验证且防止 READY_WITH_RISK 被误读；A 浪费真实反例；B 不安全；C CP7 可测试性不足。 | 影响 CR160 fail-closed 样例、CP7 verification 和 future reviewer training。 |
| CP3-DQ-CR160-005 | runtime_authorization | 是否确认 CP3 approve 不授权 Stage 5 paper/simulation/runtime 或任何外部数据/运行操作？ | Approve non-authorization boundary：CR160 只输出设计和非授权 Stage 5 view，所有 execution routes 都是 follow-up candidates。 | A. 同时设计 Stage 5 paper/simulation gate；B. 同时授权 real-data validation；C. 允许 reviewer 手工风险接受 runtime。 | 推荐方案边界清楚；A/B 都是独立高风险 CR；C 不符合 no-runtime/no-credential/no-trading policy。 | 高风险授权边界；防止 CP3 approve 被误读为 paper/simulation/live/trading/runtime 授权。 |

如果你回复 approve，表示你接受以上 5 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
