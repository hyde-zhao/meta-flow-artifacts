请审查人工门禁 `CP8-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-RELEASE-READINESS`。

checklist 路径: `process/checkpoints/CP8-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-RELEASE-READINESS.md`
自动预检结论: PASS / READY_WITH_RISK，0 个 blocker；独立 QA 未完成已作为显式、高风险、到期 waiver 路由到 CP8。

审批者摘要:
- 本次确认服务的整体目标: 确认 CR161 strategy admission evidence pipeline hardening 的设计-only 交付已满足关闭条件，并决定是否以 `READY_WITH_RISK` 关闭该 CR。
- 推荐动作: `approve`，接受 CP7 `PASS_WITH_RISK`，以 design-only `READY_WITH_RISK` 关闭 CR161；接受当前切片提供的是 evidence availability + typed_unavailable fail-closed contract，而非实际统计/经济/容量计算，并接受到期的 verifier-independence waiver。
- approve 后会发生什么: Host Orchestrator 将回填 CP8 人工结果，同步 CR161 状态为 delivered / ready_with_risk，并保留 `FU-CR161-001..006` 为候选，不自动启动。
- approve 不授权什么: 不授权代码/测试/schema/checker 实现、research-engine trial-lineage instrumentation、FDR/PBO/DSR/OOS/TCA/capacity 计算、strategy remediation、CR155 历史重建或晋级、real lake/NAS/provider/credential/broker/trading/runtime/external framework/Git remote/release execution/publish。
- 不确认会阻塞什么: 阻塞 CR161 closure；后续 computable evidence follow-up 不能以 CR161 closure 为基线。

本轮待人工决策项: 4

Context Capsule:
- `process/context/CP8-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-CONTEXT.yaml`
- read_profile: compact
- 全文档读取: `RE-20260709T222054Z0000-cr161cp8followup`、`RE-20260709T225447Z0000-b76e745c`

决策收集覆盖:
- STATE pending queue、CP7 verification result、release context、follow-up tracking 和 release docs 已扫描。
- 4 个待决策项全部纳入 CP8 Decision Brief；dispatch/fallback 可审计但独立 QA 未完成，已纳入 DQ-CP8-CR161-004。

决策分层:
- 必须用户决策: 4
- 高风险策略确认: `DQ-CP8-CR161-001` 风险接受，`DQ-CP8-CR161-002` 不授权边界，`DQ-CP8-CR161-004` verifier-independence waiver。
- agent 默认处理: release profile=compact，install/deploy/migration=N/A，follow-up candidates 只登记不启动。
- 仅审计记录: CP4/CP5/CP6 N/A、artifact repo dirty；QA fallback 已升级为显式风险接受项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| `DQ-CP8-CR161-001` | risk_acceptance | 是否接受 CP7 `PASS_WITH_RISK` 并以 design-only `READY_WITH_RISK` 关闭 CR161？ | Approve：接受设计-only 风险，确认当前交付是 fail-closed contract，不是 computable evidence implementation。 | 回退 CP3；启动实现 follow-up 后再关闭。 | 推荐方案闭合当前授权范围并保留真实边界；备选会扩大或延后当前 CR。 | 接受后 CR161 可 delivered with risk；拒绝则保持 active。 |
| `DQ-CP8-CR161-002` | runtime_authorization | 是否确认 CP8 approve 不授权任何实现、计算、数据、runtime、trading、external framework、Git remote 或 publish 操作？ | Approve deny-by-default non-authorization boundary。 | 启动单独 high-risk authorization CR；reject CP8 直到 runtime/computation evidence exists。 | 推荐方案防止 READY_WITH_RISK 被误读为运行或计算授权；备选需要更高风险门禁。 | 高风险；防止 READY_WITH_RISK 被误读为 paper/simulation/live/runtime readiness。 |
| `DQ-CP8-CR161-003` | follow_up_tracking | 是否接受 `FU-CR161-001..006` 作为候选，不在本轮启动？ | Approve：记录到 follow-up tracking 和 FEEDBACK，不创建 active CR。 | 立即提升某个 candidate；删除 selected candidate。 | 推荐方案保留后续路线且不扩大 CR161；候选不代表授权。 | 保留路线且不扩大 CR161；候选不代表授权。 |
| `DQ-CP8-CR161-004` | risk_acceptance | 是否接受 CP7 缺少独立 `meta-qa` 产物、由 host review-only fallback 完成的例外？ | 仅为本 design-only slice 接受到 `2026-10-10` 到期的 READY_WITH_RISK waiver。 | 补跑 fresh independent meta-qa review；或保持 CR161 active。 | 推荐方案保留真实 assurance 降级；备选提高独立性但延后关闭。 | HIGH；不得把 fallback 记录误称为独立 QA 完成。 |

如果你回复 approve，表示你接受 CR161 以 design-only `READY_WITH_RISK` 关闭，接受 CP7 剩余风险及到期 verifier-independence waiver，确认 deny-by-default 不授权边界，接受 6 个 follow-up candidates 只登记不启动。

不授权项:
- `approve` 不表示授权代码/测试/schema/checker、trial-lineage instrumentation、FDR/PBO/DSR/OOS/TCA/capacity 计算、real lake/NAS/provider/credential/broker/trading/runtime/external framework/Git remote/release/publish。

请只回复以下三个 exact 选项之一：
- `approve`
- `修改: <具体修改点>`
- `reject`
