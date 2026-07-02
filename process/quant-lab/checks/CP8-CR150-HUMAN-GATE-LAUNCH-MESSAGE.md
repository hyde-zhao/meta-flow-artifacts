请审查：process/checkpoints/CP8-CR150-DELIVERY-READINESS.md

自动预检结论：PASS / READY_WITH_RISK pending approval

Context Capsule：process/handoffs/NEXT-SESSION-CR150-CP8-RELEASE-READINESS-2026-07-01.md（read_profile=compact；CP8 dedicated context capsule N/A，使用 next-session handoff 作为 scoped context ref）

审批者摘要：
- 本次确认服务的整体目标：完成 CR150 multifactor framework completion 的 CP8 release readiness，决定是否接受 CP7 inline-fallback 验证风险并关闭本地 metadata linkage 切片。
- 推荐动作：approve，接受 `DEC-CR150-CP8-001`，将 CR150 CP8 结论标记为 `READY_WITH_RISK`。
- approve 后会发生什么：记录用户接受 CP7 Host Orchestrator inline-fallback waiver；CR150 本地 metadata linkage 按 `READY_WITH_RISK` 收尾；runtime/data/external operation 继续 deferred。
- approve 不授权什么：不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/Git remote/外部框架运行，不授权 publish、push、依赖安装、catalog pointer mutation 或任何外部写入。
- 不确认会阻塞什么：阻塞 CR150 CP8 closure；若用户不接受 waiver，则回退到 CP7 meta-qa/subagent verification。

本轮待人工决策项：1

决策收集覆盖：已扫描 8 个来源，发现候选问题 15 个，纳入待决策 1 个；N/A 来源 1 个，原因见 checkpoint 的 Decision Collection Coverage。

决策分层：
- 必须用户决策：1
- 高风险策略确认：0
- agent 默认处理：2
- 仅审计记录：1

如果你回复 approve，表示你接受以下 1 项推荐方案，不表示授权以下禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DEC-CR150-CP8-001 | risk_acceptance | 是否接受 CR150 CP7 由 Host Orchestrator inline-fallback 代行 meta-qa 验证的 waiver？ | 接受本次 waiver，并将 CR150 CP8 结论标记为 `READY_WITH_RISK`。 | A: 不接受 waiver，补跑 meta-qa/subagent verification；B: 接受本次 waiver 并追认 CP7 PASS 为 READY，但记录一次性例外。 | 推荐方案如实保留流程风险且不重跑已通过的静态证据；A 最严格但阻断 CR150；B 会弱化风险表达。 | 影响 CR150 是否能关闭；不影响代码实现和本地 metadata linkage 结果。 |

CP8 evidence clarifications：
- `effective_validation_mode=static-only`；CP7 的 `mixed` 标签不代表 runtime evidence。
- Phase A asset map 与 Phase B contract completion 已合并为 `MultifactorFrameworkCompletionMap`；`nodes` 提供 asset inventory，`linkage_gaps` 提供 gap-count 等价视图。
- CR150 不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/Git remote/外部框架运行。

不授权项：
- real lake read or write
- NAS sync, write or restore
- provider fetch
- catalog pointer mutation
- credential read
- QMT, MiniQMT, xtquant or gateway runtime
- simulation, paper, live, trading or broker operation
- Git remote write
- external framework clone, install or run

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

审查后请在“人工审查结果”中填写结论，也可以直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```
