请审查：process/checkpoints/CP8-DELIVERY-READINESS.md

自动预检结论：PASS
Context Capsule：process/context/CP8-DELIVERY-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）
决策收集覆盖：已扫描 6 个来源，发现候选问题 5 个，纳入待决策 3 个；N/A / 缺失来源 1 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：3

如果你回复 approve，表示你接受以下 3 项推荐方案，不表示授权以下 6 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP8-DQ-01 | risk_acceptance | 是否接受 READY_WITH_RISK 交付结论 | 接受 READY_WITH_RISK，作为当前 meta-flow 自进化交付就绪结论 | A：降级 NOT_READY 后修复；B：仅接受 artifact routing，暂缓 eval runner 扩展 | 推荐方案验证充分但需接受 residual risks；备选延迟交付或拆分范围 | 影响交付状态和风险接受记录，不影响真实发布授权 |
| CP8-DQ-02 | runtime_authorization | 是否确认 CP8 approve 不授权真实发布、外部 SaaS、凭据、trace、publish/live、production 写入或删除 artifact repo | 明确不授权，仅确认交付就绪 | A：单独 runtime_authorization CR；B：转 Spike 评估真运行 | 推荐方案权限最小；备选可增加真实证据但需要额外授权 | 防止把交付确认误解为生产操作授权 |
| CP8-DQ-03 | follow_up_tracking | 是否确认当前无 open follow-up candidate | 当前不创建后续 CR，由观察阈值触发 | A：立即创建 parser hardening CR；B：立即创建 artifact restore UX CR | 推荐方案保持台账干净；备选提前降风险但扩大范围 | 影响后续 backlog，不影响当前交付物可用性 |

不授权项：

- External SaaS execution
- Credentials
- Trace upload
- Publish / live operation
- Production target project write
- Replacing or deleting user-provided artifact directories outside the approved symlink targets

请回复以下三种之一：

- approve
- 修改: <具体修改点>
- reject
